# PortSwigger Lab: SQL Injection - Querying Database Type and Version (MySQL)

## Summary
Building on previous UNION-based labs, this lab required displaying 
the database version string by injecting a query using the MySQL-
specific `@@version` variable, after confirming the column count and 
string compatibility.

## Vulnerability
The vulnerable injection point was the product category filter, which 
reflects query results in the response without sanitizing user input.

## Steps to Reproduce
1. Confirmed the query returns two columns, both compatible with 
   string data, using:
   `' UNION SELECT 'abc','def'#`
2. Replaced the test values with the MySQL system variable that holds 
   the database version:
   `' UNION SELECT @@version, NULL#`
3. The response displayed the database version string 
   (`8.0.42-0ubuntu0.20.04.1`), confirming the database is MySQL 
   running on Ubuntu

## A debugging note (browser URL encoding issue)
Initially, submitting the payload directly in the browser's address 
bar caused an Internal Server Error, even though the payload logic was 
correct. This happened because a literal `#` character in a browser 
URL bar is treated as a page fragment/anchor and gets stripped before 
the request is even sent to the server — meaning the comment marker 
never reached the query, leaving the rest of the original SQL statement 
intact and causing a syntax error. The fix was to URL-encode `#` as 
`%23`, which preserves it as part of the actual request rather than 
letting the browser interpret it as a fragment.

## Impact
Revealing the database type and version gives an attacker valuable 
reconnaissance information — it narrows down exactly which 
database-specific syntax, functions, and known vulnerabilities apply, 
making further attacks (like extracting table names or exploiting 
version-specific bugs) significantly easier to plan.

## Fix
Use parameterized queries to prevent SQL injection at the source, and 
avoid exposing internal error messages or database details in 
application responses, since even indirect information leakage (like 
version disclosure) aids attackers in planning further exploitation.

## What I Learned
Beyond the SQL injection technique itself, this lab taught me an 
important practical lesson about how browsers handle special 
characters in URLs. A `#` typed directly into a browser address bar 
gets treated as a fragment identifier and silently stripped, which 
broke my payload even though the underlying logic was correct. 
URL-encoding it as `%23` fixed this immediately. This is a good 
reminder that testing injections directly in a browser's address bar 
can introduce encoding issues that wouldn't occur if using a proper 
intercepting tool like Burp Suite Repeater, which sends the raw request 
without this kind of browser-side interference.
