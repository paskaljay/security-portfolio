# OWASP Juice Shop: Repetitive Registration (Improper Input Validation)

## Summary
The user registration endpoint accepts a password and repeat-password 
field, intended to catch typos by requiring both to match. However, 
this matching check is only enforced in the frontend JavaScript — the 
backend API accepts registration requests even when the two password 
fields don't match, violating the DRY (Don't Repeat Yourself) 
principle by duplicating validation logic inconsistently between 
frontend and backend.

## Discovery Process
Since the registration form's UI wouldn't allow submission with 
mismatched passwords, I bypassed the frontend entirely using Burp 
Suite. After capturing a normal registration request 
(`POST /api/Users/`), I sent it to Repeater and modified the JSON body 
directly, setting `password` and `passwordRepeat` to different values, 
then sent the request directly to the server, bypassing the frontend 
form entirely.

## Vulnerability
​```json
{
  "email": "tes@dhhdk.com",
  "password": "Happy@123",
  "passwordRepeat": "Different@456",
  ...
}
​```
Despite the mismatch, the server responded with `201 Created` and 
successfully created the account, confirming the backend never 
actually validates that `password` and `passwordRepeat` match — that 
check exists only in client-side JavaScript, which is trivially 
bypassed by sending the request directly.

## Steps to Reproduce
1. Intercept a normal registration request using Burp Suite
2. Send it to Repeater
3. Modify the request body so `password` and `passwordRepeat` have 
   different values
4. Send the modified request directly to the server
5. Observe a `201 Created` success response, confirming the account 
   was created despite the mismatch

## Impact
While this specific case is relatively low severity (the repeat 
password field is a UX safeguard against typos, not a security 
boundary on its own), it demonstrates a broader pattern: any validation 
that exists only on the frontend provides no real guarantee, since 
anyone can bypass the UI entirely and interact with the API directly. 
If this pattern were applied to more security-critical validation 
(such as authorization checks or input sanitization) rather than a 
typo-catching UX feature, the consequences could be far more severe.

## Fix
Enforce password-matching validation on the backend as well as the 
frontend, following the DRY principle by ensuring the same validation 
logic (or equivalent enforcement) exists wherever the data is actually 
processed and trusted, not just where it's initially entered.

## What I Learned
This challenge was a clean, concrete demonstration of why frontend-only 
validation is never sufficient — the browser and its JavaScript are 
entirely under the user's control, so any check that matters must also 
be enforced server-side. It also reinforced the value of tools like 
Burp Suite for testing this exact class of issue: rather than trying 
to manipulate the UI directly, going straight to the API request 
itself is the most reliable way to confirm whether a given validation 
rule is genuinely enforced or just a frontend convenience.
