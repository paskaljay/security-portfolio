# PortSwigger Lab: Blind SQL Injection with Time Delays

## Summary
This lab contained a blind SQL injection vulnerability in a tracking 
cookie, with no visible output and no distinguishable error behavior — 
the only signal available was response timing. By injecting conditional 
time delays, it was possible to infer true/false answers about the 
administrator's password one character at a time, ultimately 
reconstructing the full password.

## Vulnerability
The `TrackingId` cookie value is inserted directly into a SQL query 
without sanitization. The database is PostgreSQL, confirmed by using 
`pg_sleep()`, a PostgreSQL-specific function.

## Steps to Reproduce
1. Confirmed the injection point and that conditional delays worked, 
   using:
   `' || (SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) 
   ELSE pg_sleep(-1) END FROM users)--`
   A ~10 second delay confirmed the `administrator` user exists and 
   that conditional timing could be reliably observed
2. Determined password length by testing `LENGTH(password) > N` for 
   increasing values of N, using Burp Intruder to automate the sweep:
   `' || (SELECT CASE WHEN (username='administrator' AND 
   LENGTH(password) > N) THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM 
   users)--`
3. Extracted the password character by character using `SUBSTRING()` 
   to test one character position and candidate character at a time, 
   again automated via Burp Intruder:
   `' || (SELECT CASE WHEN (username='administrator' AND 
   SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(-1) 
   END FROM users)--`
4. Logged in as `administrator` using the fully reconstructed password

## A critical debugging note (Intruder concurrency)
Initially, Burp Repeater and Intruder produced a "stream failed to 
close correctly" error when running conditional delay payloads. The 
fix was reducing Burp Intruder's resource pool from the default 10 
concurrent connections down to 1. This is an important practical 
lesson for time-based blind SQLi specifically: running multiple 
requests concurrently can cause timing measurements to interfere with 
each other, since the server may be juggling several delayed queries 
at once, making individual response times unreliable or causing 
connections to fail outright. Time-based attacks require strictly 
sequential requests to produce trustworthy timing signals.

## Impact
Time-based blind SQL injection demonstrates that even when an 
application gives absolutely no visible feedback — no data, no errors, 
no behavioral difference — an attacker can still fully extract 
sensitive data using timing alone as a covert channel. This makes it 
one of the hardest SQLi variants to detect defensively, since the 
application behaves identically from the user's perspective in every 
case except response time.

## Fix
Use parameterized queries to eliminate SQL injection at the source. 
Additionally, consider rate-limiting or monitoring for unusual patterns 
of slow requests, since time-based attacks require many sequential 
requests and produce a detectable pattern of deliberately delayed 
responses if monitored.

## What I Learned
This was the most technically demanding SQLi variant so far, since 
there's zero visible signal to work with — every previous technique 
(UNION output, error messages) gave some form of direct feedback, 
while this relies entirely on measuring time. I also learned a 
genuinely important operational lesson: automation tools like Intruder 
need to be configured correctly for the attack type being run — 
default settings optimized for speed (high concurrency) actively 
broke this technique, and reducing concurrency to 1 was necessary for 
reliable results. This is a good reminder that understanding *why* a 
tool's settings matter is just as important as knowing the injection 
technique itself.
