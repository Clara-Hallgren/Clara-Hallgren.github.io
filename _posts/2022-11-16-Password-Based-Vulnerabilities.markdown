---
layout: post
title:  "Vulnerabilities in password-based login – Portswigger Walkthroughs"
date:   2022-11-14 09:00:21 +0200
author: Clara Hällgren
---

Authentication is the process of verifying the identity of a given user or client. In this article I will 
go through Portswiggers labs on Vulnerabilities in password based logins. On a website with password based login 
they authenticate users on that they know the username and secret password combination. This could be compromised if an 
attacker gets hold of those credentials in some way, possibly via a brute force attack.

A brute-force attack is when an attacker attempts to guess credentials in bulk and most often automatically with lists 
of common passwords or usernames found on the internet. Usernames 
are easier to guess as their format often holds some sort of syntax or are publicly available. Even high privileged 
accounts often have the username admin or administrator. 

## LAB: Username enumeration via different responses

Username enumeration is a technique where you can observe changes in the responses from the website in order to verify
if a username is valid. These changes could be status codes, error messages, html or response time. In this lab the 
objective is to find a valid username from a given list of candidates, then brut-force the password to access the account. 

Step 1: I enter something random in the login box and notes the message invalid username. This is typically bad practice because
it tells an outsider which accounts that exists or not.

Step 2: Send the login request to the Intruder and paste the username list as a payload for the username input. Add "Invalid username"
of the results under options -> grep-match. Start the attack.

Step 3: Note, in the results, that one username does not display "Invalid username". We now know that this is the username of a 
valid account. The username is "adkit". 

Step 4: Do the same thing as step 2 and 3 but for the passwords instead and grep-match on "Incorrect password". As that is what
is shown instead of "Invalid username". The password is 123456. Lab solved. 

## Lab: Username enumeration via subtly different responses

This is exactly the same thing as the previous lab, except the response to look for is subtle. I tried to log in with 
random strings and got "Invalid username or passwords." so I enumerated the usernames with intruder and grep-matched that 
message. As grep-match will only mark on exact matches we will note if there is a difference with any of the usernames, and it was.
I got a hit on the username "activestat" and the message was the same but without the period at the end. With that done I could brute-force 
the password and log in. 

## LAB: Broken brute-force protection, IP block

When brut-forcing a log-in page there is a high chance, or risk depending on how you see it, that they have implemented 
some sort of protection against this. Often by locking the account after too many failed log ins or if to many attempts are
made within a time frame. This might have logic flaws and can then be bypassed, just like in this lab. The objective of the
lab is to brute force carlos password, and to our help we have our own account wiener. 

Step 1: See how the application behaves when you try to log in as carlos. On my forth attempt I get the message "You have made too many incorrect login attempts. 
Please try again in 1 minute(s).". So it's a time lock. I now log in as my own user which works. I now log out and try with carlos again, the time lock is
gone. Perfect. This is a classic IP block that blocks the ip im trying to log in from. But when I have a successful login, even with another account, the 
counter seems to reset. This means that if I can make the Intruder log in as wiener every third time, im good to go.

Step 2: Send the log-in request to Intruder. Set both username and password as payload and chose Pitch fork as attack type. 
Go to excel and write wiener and carlos on row 0 and 1 drag until carlos is repeated at least 100 times. Add peter to the 
password list before every password, I wrote a simple script for this. Make sure you are only sending one request at a time under 
resource pool and grep match on Incorrect password. Start the attack and you will find the correct password in the results.

## LAB: Username enumeration via account lock 

Another countermeasure to beat brute-force attacks is to lock the account if there are too many failed log in attempts. Although,
if the logic is flawed this can be used to enumerate passwords, which this lab is about.

Step 1: In intruder I set the attack type to cluster bomb and add a list of usernames as the first payload. I set the second payload, the passwords, to
something random like 0-9. I grep match on invalid username or password, run one request at a time and start the attack. Soon I get the result that 
apple is blocked due to too many tries, this gotta be a valid username.

Step 2. Change the username to the newly found username and choose sniper attack. Set the payload to the password list and grep-match on  
"You have made too many incorrect login attempts. Please try again in 1 minute(s).". I get three rows that does not match and one of them is my password 
which I can see in the response of that query. I can now log in and solve the lab. The account was obviously only locked if I got the password wrong, which is a logic flaw
and vulnerability. 

## LAB: Broken brute-force protection, multiple credentials per request

This lab is all about user rate limiting as it is another way to prevent brute-force attacks. This means that your IP will be blocked
if you make to many failed log-in attempts and the only way to be unblocked is waiting a certain amount of time,  manually by an administrator or manually by the user 
completing a CAPTCHA. As the limit is based on the number of HTTP requests rent it is sometimes possible to circumvent this by sending multiple guesses in a 
single request - as in this lab. The victim's username is Carlos and we got a list of passwords. 

Step 1: Try to log in and note that the login request sends the credentials in json format. Send it to the repeater. 

Step 2: In repeater replace the password with an array of the passwords in the list ["password1", "password2"] and so forth. Send the request and hope for a 302 response.

Step 3: If you got a 302 response, chose to show response in browser, copy the link and note that you are logged in as Carlos. Lab solved. 