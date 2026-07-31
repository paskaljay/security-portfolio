# PortSwigger Lab: SQL Injection UNION Attack - Retrieving Multiple Values in a Single Column

## Summary
This lab required extracting both usernames and passwords for all 
users from a separate `users` table, but unlike the previous lab, only 
one column in the query results was actually displayed. This required 
concatenating both values into a single column using the `||` operator.

## Vulnerability
The vulnerable injection point was the product category filter, which 
reflects query results in the response without sanitizing user input.

## Steps to Reproduce
1. Confirmed the column count using the technique from earlier labs 
   (testing with NULL values), and determined that only one column 
   in the query results is actually displayed on the page
2. Since two separate values (username and password) needed to be 
   extracted but only one column was visible, used the `||` 
   concatenation operator to combine both values into a single string, 
   separated by a delimiter (`~`) for readability:
   `' UNION SELECT NULL, username || '~' || password FROM users--`
3. The response displayed each user's username and password combined 
   into one field, separated by `~`, for every row in the `users` table
4. Identified the `administrator` row from the output and used the 
   corresponding password to log in

## Impact
Same as the previous lab — full credential extraction for all users, 
including administrative accounts, leading to complete account 
takeover. This variant shows that even when an application only 
displays limited output positions, an attacker can still extract 
multiple pieces of data by combining them into a single field.

## Fix
Use parameterized queries to prevent SQL injection at the source. 
Displaying fewer output columns doesn't meaningfully protect against 
this attack, since concatenation bypasses that limitation entirely — 
the only real fix is preventing the injection itself.

## What I Learned
This lab taught me the `||` concatenation operator, which lets you 
combine multiple values into a single string useful when a query 
only has one usable output column instead of one-per-value. I used a 
delimiter (`~`) between the concatenated values purely for readability, 
so the output could still be parsed visually into username and 
password. This showed me that limiting displayed output columns isn't 
a real defense against UNION based extraction, since concatenation 
easily works around that constraint.
