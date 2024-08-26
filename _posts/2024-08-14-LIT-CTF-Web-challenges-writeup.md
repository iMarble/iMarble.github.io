---
title: LIT CTF WEB JWT challenges writeup.
date: 2024-08-26
categories: [web]
tags: [web, JWT, CTF, LITCTF1]
author: 0
---

# Web/jwt-1
## Description

![alt text](https://i.imgur.com/yE4IU9R.png)

## Initial Analysis

The name of challenges in CTFs are often selected in a way to give a hint for the challenge. Same is the case for this web challenge.
Since, the name is jwt so we know we have to do something with JWT i.e Json Web Tokens to solve this challenge.

When we open the challenge website we are faced with a get fllag page which says we can't do much without logging in. So, lets log in.

![alt text](https://i.imgur.com/EhpDjbg.png)

Since we don't have a valid credentials, we'll go to to sign up page and create a new account and sign in using those credentials.
### Json Web Tokens

JSON Web Token (JWT) is an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the **HMAC** algorithm) or a public/private key pair using **RSA** or **ECDSA**.

#### Structure 
In its compact form, JSON Web Tokens consist of three parts separated by dots (`.`), which are:

- Header
- Payload
- Signature

Therefore, a JWT typically looks like the following.

`xxxxx.yyyyy.zzzzz`

You can read more about JWTs [here](https://jwt.io/introduction)

### Vulnerability

In this challenge the signatures of JWT are not verified. This means we can modify the JWT and the server will accept modified JWT without any verification.
### Solution

We'll analyze the web requests using the developer tools (ctrl+f12). You can use burp as well.

After we login with valid credentials, we are redirected to the same page. However, the login post request sets the cookies with a JWT token ( the challenge says it doesn't use cookies smh - the cookies should've been changed with authorization header which is the actual case in JWTs.)

![alt text](https://i.imgur.com/nclXgxB.png)

If we try to get the flag now, it says we are unauthorized to get the flag.

Lets copy this token and debug it using this [tool](https://jwt.io/#debugger-io)

After decoding the token, we observe that there is a admin key in json payload which is set to false.

![alt text](https://i.imgur.com/GfoS153.png)

Lets try setting this key to true and copy the modified JWT without signing it with a valid secret.

![alt text](https://i.imgur.com/ET9tU5v.png)

Now we will replace our original token in cookies with the one we generated here.

For this, we will go to storage tab in developer tools and edit the cookies.

![alt text](https://i.imgur.com/uOsqqvt.png)

Now after replacing the cookies lets try to get the flag again.

Nice, we got the flag!!!!!!
![alt text](https://i.imgur.com/hU2kag3.png)

-----------------

# Web/jwt-2

## Description
![alt text](https://i.imgur.com/hDDG6NA.png)
As the description says this challenge is similar to jwt-1. However, in this challenge we are given a typescript file too.

## Initial Analysis

The website is using JWT similar to JWT-1 challenge. However, trying to modify the JWT and using it without a valid signature doesn't give us the flag, instead it still says we are unauthorized to get the flag.

Lets view the typescript file to see if we can find more information there.

After opening the file, we immediately notice a variable named "jwtSecret". Some juicy info, nice.

![alt text](https://i.imgur.com/VlIXrak.png)

From the code, we see that jwtSecret is used for signing the JWT.

![alt text](https://i.imgur.com/CWlyrM2.png)

Then in the /flag endpoint we see the authorization check.
1. First the cookies are checked if they contain the token.
2. If cookies have the token, it is split into three parts i.e header, payload and signature
3. The payload is base64 decoded in the next step.
4. Signatures are verified using the jwtSecret.
5. If user is admin, flag is given.

## Vulnerability

Sensitive information i.e jwtSecret disclosure. It can lead to generation of any valid signatures using the secret and bad actor can generate valid token for any user.

## Solution

- We go to the JWT debugging website [here](https://jwt.io/#debugger-io)
- Edit the token and sign it using the secret
- Edit the cookies in the website and replace it with edited token
- Got the flag

![alt text](https://i.imgur.com/n3VLiFo.png)