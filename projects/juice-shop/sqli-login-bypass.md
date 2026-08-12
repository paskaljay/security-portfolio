# OWASP Juice Shop: SQL Injection Login Bypass

## Summary
Juice Shop's login form is vulnerable to SQL injection, allowing 
authentication bypass without valid credentials. By injecting SQL 
logic into the email field, it's possible to log in as the first user 
in the database — which turned out to be the admin account.

## Discovery Process
This was found through unguided exploration (no walkthrough or hints), 
as part of practicing bug bounty-style hunting on a real-feeling 
target application.

Initial attempt: tried injecting `' OR 1=1--` directly into the URL, 
which had no effect — just redirected to the homepage. This was 
because Juice Shop's login form submits data via a POST request, not 
URL parameters, meaning the URL itself was never actually part of the 
vulnerable query. This was a useful reminder that not every injection 
point works the same way as PortSwigger's URL-based labs — the actual 
form fields needed to be used directly.

## Steps to Reproduce
1. Navigate to the login page via the account icon
2. In the **email** field, enter:
   `' OR 1=1--`
3. In the **password** field, enter any value (e.g. `test123`)
4. Submit the form
5. Successfully logged in as `admin@juice-sh.op`, without knowing the 
   actual admin password

## Impact
This is a critical vulnerability complete authentication bypass 
granting access to whichever account the backend query happens to 
return first, in this case the admin account. An attacker gains full 
admin access without any credentials at all.

## Fix
Use parameterized queries so user input is never interpreted as 
executable SQL syntax, preventing the injected logic from altering the 
query's intended behavior.

## What I Learned
This was my first fully unguided vulnerability discovery, without any 
lab instructions telling me where to look or what technique to use. 
The initial failed URL-based attempt was actually a valuable lesson: 
real-world applications don't always expose injection points the same 
way structured labs do, and recognizing *how* a form submits data 
(POST vs URL parameters) matters before assuming an injection attempt 
has failed. This reinforced that the underlying SQL injection 
technique from PortSwigger's login bypass lab transfers directly to 
real applications, but finding the correct injection point requires 
more investigation without guided hints.
