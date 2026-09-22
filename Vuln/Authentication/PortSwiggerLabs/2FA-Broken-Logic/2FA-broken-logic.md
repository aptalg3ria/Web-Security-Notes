# Lab: 2FA Broken Logic

## Objective

Access the victim's account by abusing a flaw in the 2FA verification logic and the lack of brute-force protection.

---

## 1. Login With My Account

I started by logging in with my own account:

```text
wiener:peter
```

After successfully logging in, the application asked me to enter the 2FA code.

I activated **Burp Proxy** and intercepted the request.


## 2. Manipulating the `verify` Parameter

While looking at the request, I noticed a `verify` parameter:

```http
verify=wiener
```

Since I already knew the victim's username was:

```text
carlos
```

I changed the parameter to:

```http
verify=carlos
```

Then I sent the modified request.

### Screenshot

> **[Screenshot — Modified `verify` parameter]**


VerifyCarlos.png


The application accepted the request and returned the 2FA verification page normally.

This was the important discovery.

The application did not properly verify that the user who authenticated with the password was the same user specified by the `verify` parameter.

I was authenticated as:

```text
wiener
```

but I could make the application continue the 2FA process for:

```text
carlos
```

### Logic flaw

```text
Authenticated user
       |
       v
    wiener
       |
       |  verify=carlos
       v
2FA verification for Carlos
```

The application was trusting a client-controlled parameter to determine whose 2FA verification should be performed.

---

## 3. Testing Failed 2FA Attempts

Now I needed Carlos's 2FA code.

The 2FA code contained **4 digits** .

So the possible values were:

```text
0000 → 9999
```

Before brute-forcing the code, I tested whether the application would block repeated failed attempts.

I submitted several random codes.

The application continued accepting attempts and did not appear to apply an effective lockout or rate limit.

### Screenshot

> **[Screenshot  — Multiple failed 2FA attempts without blocking]**


no-rate-limit.png


This meant that brute-forcing the 4-digit code was possible.

---

## 4. Brute-Forcing the 2FA Code

I used **Burp Intruder** to automate the requests.

Instead of running all 10,000 possibilities in one attack, I divided the range into several smaller attacks.

For example:

```text
Attack 1: 0000 → 1000
Attack 2: 1001 → 2000
Attack 3: 2001 → 3000
...
```


I then monitored the responses and filtered them by status code:

```text
302
```


I identified the 4-digit code that produced the successful response and submitted it.

The application accepted the code and I gained access to Carlos's account.

### Screenshot

> **[Screenshot  — Successful login / lab solved]**


lab-solved.png


---

## Vulnerability Chain

The attack required combining two weaknesses.

### 1. Broken 2FA logic

The application trusted the client-controlled:

```http
verify=
```

parameter to determine which user's 2FA process was being verified.

This allowed me to switch from:

```text
verify=wiener
```

to:

```text
verify=carlos
```

without having authenticated as Carlos with the first factor.

### 2. Missing brute-force protection

The application did not effectively restrict repeated 2FA attempts.

Because the code was only 4 digits, there were only:

```text
10,000
```

possible combinations.

---

## Attack Flow

```text
Login as wiener
       |
       v
wiener:peter
       |
       v
Reach 2FA verification
       |
       v
Change verify=wiener
       |
       v
Change to verify=carlos
       |
       v
Reach Carlos's 2FA verification
       |
       v
No effective brute-force protection
       |
       v
Brute-force 4-digit code
       |
       v
Find valid code
       |
       v
Access Carlos's account
```

---

## Key Takeaway

The important part of this lab was not simply brute-forcing a 4-digit code.

The real issue was the **logic flow between the first authentication step and the 2FA step**.

The application allowed the client to influence which user's 2FA process was being verified. Once that logic flaw was combined with the lack of effective brute-force protection, the short 4-digit code became practically guessable.


