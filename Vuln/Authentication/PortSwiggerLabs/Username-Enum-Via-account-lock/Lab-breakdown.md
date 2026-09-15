# Username enumeration via account lock
*Lab*

## Overview

This lab was about finding a valid username by abusing the account lockout mechanism. The application temporarily blocks login attempts after several failed attempts for the same username. This behavior can be used to distinguish a valid username from invalid ones.

## Username Enumeration

First, I used Burp Suite Intruder to brute-force the username parameter.

I sent multiple login attempts for each username within a short period of time. After around 4 attempts, I noticed that one of the responses had a significantly different response length.

The different response contained a message saying that too many login attempts had been made and that I should try again later. This was important because it changed the normal response behavior.

For example:

```
Normal response → Invalid username or password
Different response → Too many attempts, try again later
```

This difference allowed me to identify a username that triggered the account lock. After testing the usernames, I found:

```
Username: ar
```

So the account-lock behavior leaked information about whether a username existed.

## Password Brute Force

After identifying the valid username, I moved to the password.

I used Burp Suite Intruder again to brute-force the password with the username fixed to:

```
ar
```

This time, I compared the response lengths of the different requests. Most of the incorrect passwords returned the same error response, so they had approximately the same response length. One response was different and had a smaller response length. That response did not contain the usual incorrect-password message, which indicated that the login attempt was successful.

Therefore, by comparing the response lengths, I was able to identify the correct password.

## Logic Flow

```
Brute-force usernames
        ↓
Send several attempts for each username
        ↓
Account lock response appears
        ↓
Different response behavior
        ↓
Identify valid username
        ↓
Fix the username
        ↓
Brute-force passwords
        ↓
Compare response lengths
        ↓
Find the response without the normal error
        ↓
Identify the valid password
```

## What I Learned

The important part of this lab was not simply the brute force itself, but the information leaked by the application's responses. Even when an application does not directly say whether a username exists, differences in response content or response length can reveal information about the account.

The account lockout mechanism was intended as a protection against brute force, but because its behavior was different for existing accounts, it actually helped with username enumeration.tr((((((((h ugtr(t'
