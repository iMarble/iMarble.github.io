---
title: SEKAI CTF APK reversing challenge writeup.
date: 2024-08-26
categories: [android]
tags: [android, reversing, CTF, SEKAICTF]
author: 0
---

# CrackMe - Reversing
In this CTF, I played with team indus. I solved the crackme challenge of reversing. This challenge was related the reverse engineering an android apk reversing.

## Description

![alt text](https://i.imgur.com/kZEDhyC.png)

## Initial Analysis

We are given an apk file and nothing else. It is mentioned that it is safe to use the app on personal phone. So, lets install the app on our android/emulator to analyze what the app does.

On opening the app, there is a login screen.

![](https://i.imgur.com/DIXwMRl.png)

okay so we have to crack the login/password or bypass this screen.

## Reversing the app

Lets try reversing the app using apktools. You can install apktools in your system using the following guide. 

Now I used this command to decompile the apk
```sh
apktool d CrackME.apk
```

Apktool decompiled the apk into different files. The dex classes are decompiled to smali code. You can read more about smali code here.

After analyzing the decompiled files, we found that this is a react native application. This can be confirmed by the presence of following things

1. React native reference in `AndroidManifest.xml` files

![](https://i.imgur.com/KLRsmg6.png)

2. Presence of ``index.android.bundle`` file in assets folder.


## Bundle file

index.android.bundle is a special file in react native based decompiled apps that contains the actual react code for the application. However, this file is difficult to read due to being obfuscated. I used [react-native-decompiler](https://www.npmjs.com/package/react-native-decompiler) to decompile it into multiple JavaScript files.

```sh
npx react-native-decompiler -i app.zip/assets/index.android.bundle -o output/
```

Because this bundle is heavily packed, there is a lot of code that serves no use to us, and names are mostly lost. But the best bet is to simply take a quick glance at all the files to see if you recognize anything. **Searching for strings** is also very useful if you know some strings when you start the app in an emulator.

I searched for the following strings
	`admin` `login` `password` 

after searching for these strings I found this part of code interesting
```js

if ('admin@sekai.team' !== t.state.email || false === e.validatePassword(t.state.password)) console.log('Not an admin account.');

else console.log('You are an admin...This could be useful.');

var s = module488.getAuth(n);

module488.signInWithEmailAndPassword(s, t.state.email, t.state.password).then(function (e) {

t.setState({

verifying: false,

});

var n = module486.ref(o, 'users/' + e.user.uid + '/flag');

module486.onValue(n, function () {

t.setState({

verifying: false,

});

t.setState({

errorTitle: 'Hello Admin',

errorMessage: "Keep digging, you're almost there!",

});
```

This code does the following:
1. It checks if the username is `admin@sekai.team`
2. It calls the validate function to validate the password. This is interesting function. lets analyze it.

### Validate Password Function

```js
e.validatePassword = function (e) {

if (17 !== e.length) return false;

var t = module700.default.enc.Utf8.parse(module456.default.KEY),

n = module700.default.enc.Utf8.parse(module456.default.IV);

return (

'03afaa672ff078c63d5bdb0ea08be12b09ea53ea822cd2acef36da5b279b9524' ===

module700.default.AES.encrypt(e, t, {

iv: n,

}).ciphertext.toString(module700.default.enc.Hex)

);

};
```

This function does the following checks
1. It checks if password is 17 characters long.
2. It returns true or false value.

The return statement is interesting
- It compares the given hex value with the `AES encrypted password`

The hex value is `03afaa672ff078c63d5bdb0ea08be12b09ea53ea822cd2acef36da5b279b9524`

Now we need the private key and IV to decrypt the password. It is present in module456 according to our decompiled code.
So, lets view the module 456

```js
var _ = {

LOGIN: 'LOGIN',
EMAIL_PLACEHOLDER: 'user@sekai.team',
PASSWORD_PLACEHOLDER: 'password',
BEGIN: 'CRACKME',
SIGNUP: 'SIGN UP',
LOGOUT: 'LOGOUT',
KEY: 'react_native_expo_version_47.0.0',
IV: '__sekaictf2023__',
};

exports.default = _;
```

Here we got the private key and IV value. Lets decrypt the AES encrypted password using these values.

I used cyberchef to decrypt this. [Link](<https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'UTF8','string':'react_native_expo_version_47.0.0'%7D,%7B'option':'UTF8','string':'__sekaictf2023__'%7D,'CBC','Hex','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=MDNhZmFhNjcyZmYwNzhjNjNkNWJkYjBlYTA4YmUxMmIwOWVhNTNlYTgyMmNkMmFjZWYzNmRhNWIyNzliOTUyNA>)

Nice we got the password. Lets login into the app.

### Login in the app

After we login, we get the following message, but no flag :(

![](https://i.imgur.com/WE12jRW.png)

Hmmm, we have to keep digging, Maybe intercept the HTTPS traffic?

## Intercepting HTTPs

We can intercept network traffic on android using different proxies such as burp.
For interception of HTTPs you will need to add your custom certificate to root store of device.
However, on modern android versions it is difficult to inject your certificate as root certificate. 

You can read more about sniffing HTTPs traffic and injecting certificates from this wonderful resource. [HTTP Toolkit](https://httptoolkit.com/blog/android-14-install-system-ca-certificate/)

Fortunately I have a rooted phone. So, I used HTTP Toolkit to analyze the HTTPs traffic. 

This toolkit automatically inject root certificate and help you to intercept HTTPs traffic easily.

After setting up HTTP toolkit, I logged in to the app and boom we got connected to a websocket. We can see this in the toolkit and there the flag is in one of the requests.

![](https://i.imgur.com/Q0mMoz3.png)

![](https://i.imgur.com/W5ULboA.png)