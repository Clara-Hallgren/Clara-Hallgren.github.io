---
layout: post
title:  "The Basics of Cross-Site Scripting"
date:   2022-08-30 09:00:21 +0200
author: Clara Hällgren
cover_image: xss_header.png
---

Cross-site Scripting, or `XSS`, is a very common vulnerability in web applications,
probably the most frequently occurring web security vulnerability.
Just last month a pair of XSS-vulnerabilities in Google Cloud and Play could have allowed hackers to hijack accounts [1]. 
I will do my best to explain what XSS is and how it arises in a simple way. Bear with me here, 
this is a big one, and I will probably post more advanced articles later on.   

## Cross-Site Scripting
XSS is a client-side vulnerability causing a website to return malicious JavaScript to users. 
When it executes inside the user’s browser, the user-application interaction can be fully compromised.
This can allow an attacker to act as the victim user and carry out any actions that they are able to perform.
There are three main types of XSS-attacks called Reflected, Stored and DOM-based XSS. In this article I will cover 
the two first types.

### Reflected Cross-Site Scripting
Reflected XSS is caused by an application that includes data from an HTTP-request within the `immediate` response
in an unsafe way. Imagine that you are using a search-function of a website and notice that the search-term is reflected
on the page, maybe in the form of “Search result for: <your search term>”. This might make it possible to input 
JavaScript in the search-parameter and have that executed in the browser. To prove the concept, you insert a harmless 
script like the alert-function as the search-term and if it executes in your browser – you found yourself a reflected 
XSS vulnerability. 

#### Let me demonstrate on a very simple case using burp suite. 
    1. I send a HTTP request with the search parameter set to a random string, in this case with URL encoding. 

![Image](/assets/img_5.png)

    2. The response shows that the search parameter is directly reflected in the HTML. 

![Image](/assets/img_6.png)

    3. Now it's time to prove the concept and insert the harmless alert funtion as the serach parameter. 

![Image](/assets/img_7.png)

    4. And voila! The script is reflected and executes in the browser. 

![Image](/assets/img_8.png)

Now that there is a proved reflected XSS vulnerability we can use this to insert Scripts that steal data, like session 
tokens or user data. Then, we have to make the victim request our Script-injected URL to execute the malicious code 
in their browser. I might send the link to the victim or place it on a website that I control and lure my victim to it.
The need for a delivery method is what makes the reflected XSS generally less harmful than other types. Although, if I 
succeed, I can typically fully compromise the user interaction with the application.

`NOTE:`The example above will (hopefully) not be found in the wild. Most applications use counter measures against XSS,
like escaping script characters and so forth. 

### Stored Cross-Site Scripting
Stored XSS is caused by an application that includes data from an untrusted source within its `later` HTTP responses in an 
unsafe way. Notice the difference from reflected XSS? These do not need to be externally delivered to the victim, 
these are stored in the vulnerable application itself which in general makes them more harmful. Maybe a website allows 
users to openly comment on posts and an attacker can submit a malicious script in a comment. Then, every user loading that 
comment by viewing the post, will have that script executed in their browser. The method is the same as for the reflected 
example above, just in the comment section instead.

## Notes
In the example above I use Portswiggers lab on XSS [2]. They have a comprehensive section on XSS with loads of vulnerable
labs to put your skills to the test [3].

## References

[1] The Daily Swig | Cybersecurity news and views. 2022. XSS vulnerabilities in Google Cloud, Google Play could lead to account hijacks. [online] Available at: <https://portswigger.net/daily-swig/xss-vulnerabilities-in-google-cloud-google-play-could-lead-to-account-hijacks> [Accessed 30 August 2022].

[2] Academy, W. and scripting, C., 2022. What is cross-site scripting (XSS) and how to prevent it? | Web Security Academy. [online] Portswigger.net. Available at: <https://portswigger.net/web-security/cross-site-scripting> [Accessed 30 August 2022].

[3] Academy, W. and scripting, C., 2022. Lab: Reflected XSS into HTML context with nothing encoded | Web Security Academy. [online] Portswigger.net. Available at: <https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded> [Accessed 30 August 2022].