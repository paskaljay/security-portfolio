# PortSwigger Lab: SQL Injection UNION Attack - Retrieving Data from Other Tables

## Summary
Combining column count and data type techniques from previous labs, 
this lab required extracting actual data (usernames and passwords) 
from a separate `users` table using a full UNION-based SQL injection 
attack, then using that data to log in as the administrator.

## Vulnerability
The vulnerable injection point was the product category filter 
(accessible via category links such as Clothing, Shoes, and 
Accessories), which reflects query results in the response without 
sanitizing user input.

## Steps to Reproduce
1. Initially attempted the injection on the account/profile URL 
   parameter, which did not work this wasn't the actual vulnerable 
   injection point for this lab
2. Identified the correct injection point: the category filter 
   parameter, accessed through category links (e.g. Clothing, Shoes, 
   Accessories)
3. Using the column count (2) and known string-compatible column 
   position from previous labs, constructed the following payload:
   `' UNION SELECT username, password FROM users--`
4. The response returned all usernames and passwords from the `users` 
   table, including the `administrator` account and its password
5. Used the retrieved administrator credentials to log in successfully

## Impact
This is a critical vulnerability — an attacker can extract entire 
tables of sensitive data, including credentials for privileged 
accounts, directly through the application's front-end. This can lead 
to full account takeover, including administrative access, without any 
need for phishing or credential guessing.

## Fix
Use parameterized queries to eliminate SQL injection at the source. 
Additionally, avoid reflecting raw query results directly into 
application responses in ways that could expose unintended data, and 
ensure passwords are hashed (not stored in plaintext) so even a 
successful extraction doesn't yield directly usable credentials.

## What I Learned
This lab tied together everything from the previous three labs: 
determining column count, identifying which column accepts string 
data, and finally using both to extract real data from a completely 
different table than the one the application intended to query. I also 
learned firsthand that identifying the correct injection point isn't 
always obvious — my first attempt targeted the wrong parameter entirely, 
which reinforced that real-world testing often involves ruling out 
incorrect assumptions before finding the actual vulnerable input. This 
feels like the first lab where the individual techniques came together 
into a genuinely complete, real attack chain.
