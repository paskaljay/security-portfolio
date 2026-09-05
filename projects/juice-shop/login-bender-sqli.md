# OWASP Juice Shop: Login Bender (SQL Injection)

## Summary
Similar to the original Login Admin finding, this challenge required 
logging in as a specific known user (Bender) using SQL injection, but 
this time targeting the user by their known email address directly 
rather than using a generic "match everything" condition.

## Discovery Process
Since the original login bypass (`' OR 1=1--`) returns whichever user 
happens to be first in the query results, a more targeted approach was 
needed to log in as a *specific* user. Using Bender's known email 
address directly in the injection payload achieves this precisely.

## Vulnerability
This uses the same underlying SQL injection vulnerability documented 
in the Login Admin finding the login query concatenates user input 
directly into a SQL string without sanitization.

## Steps to Reproduce
1. Navigate to the login page
2. In the email field, enter:
   `bender@juice-sh.op'--`
3. In the password field, enter any value
4. Submit — the `'--` closes the email string and comments out the 
   password check, logging in as Bender specifically without needing 
   to know his actual password

## Impact
This confirms the login bypass vulnerability can be precisely targeted 
at any specific user whose email address is known (rather than only 
being useful for grabbing an arbitrary "first" user), which is a more 
realistic and dangerous real world attack scenario an attacker 
targeting a specific known individual (e.g., a company executive) 
rather than just any account.

## Fix
Identical to the Login Admin fix: use parameterized queries so user 
input is never interpreted as executable SQL syntax, regardless of 
which specific email is targeted.

## What I Learned
This reinforced that the same vulnerability can be exploited in 
different ways depending on the attacker's goal broad access (any 
user) versus precise, targeted access (a specific known individual). 
Both stem from the identical root cause and identical fix, but the 
targeted version is arguably more dangerous in a real attack scenario, 
since it doesn't rely on the current row-ordering of a database to get 
what you want.
