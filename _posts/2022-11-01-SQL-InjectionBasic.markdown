---
layout: post
title:  "SQL Injection Basics (Apprentice) - Portswigger Walkthroughs"
date:   2022-11-02 09:00:21 +0200
author: Clara Hällgren
---

SQL-injection, or SQLi, might be one of the most well-known vulnerabilities out there. The sneaky thing about these are that SQLi vulnerabilities can just as well be 
caused by coders that frankly don't know what they are doing. In this article, I will go through the 
basics of SQLi and demonstrate the vulnerability by walk-throughs of the related labs from Portswigger.

## SQL Injection 
SQL injection is a web security vulnerability where an attacker finds a way to modify or interfere with database queries. 
This can allow an attacker to read data from a database, and sometimes even modify or delete its content. To truly understand and master exploiting SQLi you need 
to have a basic to intermediate understanding of how a database is built, managed and queried. And if you don't, this will probably be a quite boring article for you.

### LAB 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
We know from the description that this lab contains a SQL injection vulnerability 
in the product category filter. So, obviously, when we open the lab we firstly want to play around with this filter. I click on the “Gift” filter and see that the query is visible in the URL.

    https://<your-lab-id>.web-security-academy.net/filter?category=Gifts

From the lab description we already know that this is vulnerable. We also know that the objective is to prove this. 
Therefore, I add a simple query to the URL (category=Gifts'OR+1=1--) which solves the lab. So why did I do that?  

    https://<your-lab-id>.web-security-academy.net/filter?category=Gifts'OR+1=1--

The OR 1=1 is a powerful “tool” to learn. What I can figure out from the 
“Category=Gifts” part of the query is that it "probably" is a query that retrieves 
all entries that have the attribute Category set to Gifts. It probably looks 
something like this: 

    SELECT * FROM table_name WHERE Category=”Gifts” (Where “Gifts” is the user input).

Another key thing to notice here, is the double dash at the end of my manipulated query (--). In SQL this is a comment indicator, just like // is in Java or # is in bash. By adding this, I tell the backend that the rest of the SQL query (if there is any) is a comment and will not be queried to the database. Doing that I can reveal data I did not know existed or remove conditions that otherwise would ruin the injection. 

`Note`: Manual SQL injection is manly about understanding the database used, how the code queries data and what you can retrieve. I will say it again, gaining knowledge about different databases and queries is important for mastering SQL-injection. 

### LAB 2: SQL injection vulnerability allowing login bypass
In the previous lab we took advantage of a vulnerability in a WHERE clause 
of a query. In this lab we will try to subvert the application logic. We know 
from the instructions that the vulnerability is found in the login function and 
the objective is to log in as an administrator. 

The image below shows the payload of the intercepted request when I try to log in. 

![Image](/assets/SQLi_Basics1.png)

Just like in the previous lab I will assume the query to the database 
that happens in the backend after this request. It probably looks something 
like this:

    SELECT * FROM users where USERNAME = test AND password = test

And if the count of the result comes back as 1, or something similar, we are 
allowed to log in. This means that if we can make the query return 1 entry 
from the database on that request, we will be able to log in as the user we 
set as username (if it exists in the database of course). Remember the 
double-dash sequence from the previous lab? In other words, lets use the 
double-dash to remove the AND password part of the query which successfully 
solves the lab. 

![Image](/assets/SQLi_Basics2.png) ![Image](/assets/SQLi_Basics3.png)