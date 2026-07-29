# PortSwigger Lab: SQL Injection - Login Bypass

## Summary
The login function is vulnerable to SQL injection, allowing 
authentication bypass without knowing a valid password. By commenting 
out the password check in the underlying query, it's possible to log 
in as any known user.

## Vulnerability
The login query likely follows this pattern:
​```sql
SELECT * FROM users WHERE username = 'input' AND password = 'input'
​```
Since the username field isn't sanitized, an attacker can inject SQL 
syntax to alter the query's logic.

## Steps to Reproduce
1. Identify a valid, existing username (in this case, found via an 
   "accounts" page that exposed a username, `administrator`)
2. In the login form's username field, enter:
   `administrator'--`
3. Enter any arbitrary value in the password field - this is required 
   client-side by the form's JavaScript validation but is never 
   actually checked by the backend once the query is manipulated
4. Submit the form — the query executes as if only checking for the 
   username, since the comment sequence removes the password condition 
   entirely
5. Successfully logged in as `administrator`

## Impact
This is a critical vulnerability — it allows complete authentication 
bypass for any known username, including administrative accounts. An 
attacker doesn't need to know or guess a password at all, only a valid 
username, which is often easy to find or guess (e.g. "admin", 
"administrator").

## Fix
Use parameterized queries so user input is never treated as executable 
SQL. Additionally, avoid exposing valid usernames anywhere in the 
application (like an accounts page) since username enumeration makes 
this kind of attack much easier.

## What I Learned
This lab showed me that the same comment-based technique from the 
previous SQLi lab can be reused for a very different purpose — instead 
of bypassing a filter condition, it can bypass authentication entirely 
by removing the password check from the query. I also noticed that 
client-side form validation (requiring a password field to be filled) 
has no bearing on backend security — the frontend and backend need to 
be treated as two completely separate trust boundaries. This makes we 
think that many login systems are less secure than they appear if 
they rely on any client-side checks for security-critical logic.
