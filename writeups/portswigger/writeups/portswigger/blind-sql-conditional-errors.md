# PortSwigger Lab: Blind SQL Injection with Conditional Errors

## Summary
This lab contained a blind SQL injection vulnerability in a tracking 
cookie, where the application gave no visible query output — only a 
custom error message if the injected query caused a database error. 
This required inferring data one true/false condition at a time, using 
error presence/absence as the only signal, ultimately extracting the 
administrator's password character by character.

## Vulnerability
The `TrackingId` cookie value is inserted directly into a SQL query 
without sanitization. The database is Oracle, confirmed by the use of 
the `dual` table and `TO_CHAR()` function, both Oracle-specific.

## Steps to Reproduce
1. Confirmed the injection point using a single quote in the cookie 
   value, which caused a 500 error, while an unrelated double quote 
   returned a normal 200 response
2. Verified subqueries could execute without error using:
   `'|| (SELECT ' ' FROM dual) || '`
3. Confirmed the `users` table exists:
   `'|| (SELECT ' ' FROM users WHERE rownum=1) || '`
4. Confirmed an `administrator` user exists in that table:
   `'|| (SELECT ' ' FROM users WHERE username='administrator') || '`
5. Built a conditional error payload using `CASE WHEN`, which only 
   triggers a divide-by-zero error if the condition is true:
   `'|| (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE ' ' END FROM 
   users WHERE username='administrator') || '`
   This returned a 500 error, confirming the technique works when the 
   condition is true
6. Used the same technique to determine password length, testing 
   `LENGTH(password) > N` for increasing values of N until the 
   response switched from error to normal — narrowing down the exact 
   password length (found to be 20 characters) using Burp Intruder to 
   automate testing across a range of lengths
7. Extracted the password character by character using 
   `SUBSTR(password, position, 1) = 'character'`, using Burp Intruder's 
   cluster bomb attack type to test combinations of position and 
   candidate characters automatically, watching for which combination 
   returned an error (indicating a correct character match)
8. Reconstructed the full password from the confirmed characters and 
   logged in as `administrator`

## Impact
Blind SQL injection is just as dangerous as visible UNION-based 
injection, even though no data is directly displayed. An attacker can 
still fully extract sensitive data — including admin credentials — by 
asking the database a series of true/false questions and inferring the 
answers from indirect signals like error messages, response timing, or 
subtle content differences. Automating this with tools like Burp 
Intruder makes full data extraction practical even at scale.

## Fix
Use parameterized queries to eliminate SQL injection at the source. 
Additionally, avoid exposing custom error messages that reveal whether 
a query succeeded or failed — generic, uniform error handling removes 
the very signal blind SQL injection relies on.

## What I Learned
This lab was a significant step up from the UNION-based labs, since 
there was no visible data to confirm progress against — only an 
indirect error/no-error signal. I learned how to use `CASE WHEN` 
combined with a deliberately error-causing expression (divide by zero) 
to convert a true/false condition into an observable signal. I also 
learned to use Burp Intruder practically for the first time: first 
with a simple payload list to narrow down password length, then with a 
cluster bomb attack to test two variables simultaneously (character 
position and candidate character) for the full character-by-character 
extraction. This made clear why blind SQL injection, despite being 
slower and more indirect than UNION attacks, is just as serious a 
vulnerability in practice, especially when automated.
