---
layout: post
title:  "SQLi Union Attacks – Portswigger Walkthroughs"
date:   2022-11-02 09:00:21 +0200
author: Clara Hällgren
---

SQL injection is a web security vulnerability where an attacker finds a way to modify or 
interfere with database queries. This can allow an attacker to read data from a database, 
and sometimes even modify or delete its content.  In this article, I will continue with 
walkthroughs of Portswiggers labs to explain the concept of SQLi Union attacks. If you are 
new to SQL injection, I would recommend reading my previous article on the subject first.

## SQLi Union Attack
Databases are usually made of a number of tables with different data. 
For an online store we might have a table named products, one named users and another named discounts. Tables allow for UNION commands where we can fetch data from several tables at once. In cases where the results of and SQL query are returned in the response, we can in try to leverage that to get data from other, and maybe far more interesting, tables. We do this with a ‘UNION’ command.

## The Union Command
The union command allows for additional SELECT queries that can retrieve data from several tables at once. This is a great function for database managers that wants to fetch data from multiple tables in one request. It can also be used by attackers that has found a SQLi vulnerability. Lets say we have the following request to the database:
SELECT product_name, product_price FROM products
This would give us all entries from the products table in one column. Now, we try to add an additional SELECT query:
SELECT product_name, product_ description FROM products UNION SELECT username, password FROM users
This will return a result set with two columns. The first column will contain the product names and description while the other column will contain the usernames and passwords. For this to work, two criteria’s must be met. The individual queries must return the same number of columns from the database, in this case product name / description and username / password. The second criteria is that the datatypes in the SELECT statements must be compatible with the data you want to retrieve.  This means that we somehow need to figure out the number of columns returned and the datatypes. I will do my best to explain this in the four upcoming walkthroughs. 

## SQL injection UNION attack, determining the number of columns returned by the query
The objective of this lab is to determine the number of columns by a UNION attack that returns and additional row containing null values. Here is how I did it: 
First, I determine the number of columns that are returned for the first criteria. I do this by adding ‘ORDER BY X-- to the query, where X is a number that I increment by 1 for each try. What I am looking for here is some type of error message displayed on the site due to the database generating an error. When I get up to number 4, I get an Internal Server Error, which means that 3 is the magic number. Note that all vulnerable applications don’t show an error message even if it is thrown.
Second, I compose a UNION query requesting null values because they are compatible with most datatypes (the second criteria). And because 3 is the magic number in this case I will need to request 3 NULL values like this:
    
    ‘UNION SELECT NULL,NULL,NULL—

Which solves the lab! 

## SQL injection UNION attack, finding a column containing text
In this lab the objective is to figure out which requested column that can display text. This is a great practice of the second criteria for SQL union attacks but note that it is not a great example of how to exploit this in the wild. Here we need to extract a random value and make it appear with the query result, lets get to it! My random value is rzzH7w. 
1. I start by determining the number of columns just like in the last lab with ORDER BY X and I get 3 as the magic number again. 
2. Now I need to figure out which of the three NULL values that is compatible with String/varchar. I add my random value to the first null which generates an error so I move on to the next one where I find my luck (‘UNION SELECT null,’rzzH7w’,null—)
3. Lab solved.

## SQL injection UNION attack, retrieving data from other tables

Now that we have determined the number of columns and which columns that can hold string data we have completed the two 
criteria’s for UNION attacks and can move on to accessing something of interest.  In the description of the lab we get 
to know that there is a table named user that has the columns username and password. Without this information we would 
have had to guess or find a way to examine the database, which we will come back to in the next article, but for this lab we get that information for free. 
1. I use the ORDER BY method to learn that I have to query two columns in my UNION statement. 
2. I use the UNION SELECT null,null method to learn that both can hold string data. See image below.
![Image](/assets/SQL-Union/img.png)
3. I create a query that retrieves the “user” table that was stated in the lab description. In the returned result I can see the administrators' login credentials 

![Image](/assets/SQL-Union/img1.png)

## SQL injection UNION attack, retrieving multiple values in a single column

This lab has the same objective as the previous lab except that we will need to retrieve the 
data we want to a single column. It is basically the same thing as in the previous example 
except from how we write the query. In the previous lab we wrote the query like this: 

    ‘UNION SELECT username, password FROM users

Now, because only the second column can hold a string we write the following 

    ‘UNION SELECT NULL, username ||´-´|| password FROM users

![Image](/assets/SQL-Union/img2.png)