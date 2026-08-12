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

## Source Code Analysis (Find It / Fix It Challenge)

After exploiting the vulnerability manually, Juice Shop's built-in 
"Find It / Fix It" challenge revealed the actual vulnerable source code:

```javascript
models.sequelize.query(`SELECT * FROM Users WHERE email = 
'${req.body.email || ''}' AND password = 
'${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, 
{ model: UserModel, plain: true })
```

The vulnerability came from directly embedding `req.body.email` (raw 
user input) into the SQL string using JavaScript template literals, 
with no sanitization. This matches exactly what I exploited: injecting 
`' OR 1=1--` into the email field broke out of the intended string and 
altered the query logic to always match, returning the first user in 
the table (the admin account).

## The Fix

```javascript
models.sequelize.query(`SELECT * FROM Users WHERE email = $1 AND 
password = $2 AND deletedAt IS NULL`,
  { bind: [ req.body.email, security.hash(req.body.password) ], 
  model: models.User, plain: true })
```

The fix replaces direct string concatenation with a parameterized 
query, using `$1` and `$2` as placeholders and passing the actual 
values through a separate `bind` array. This ensures user input is 
always treated as pure data by the database driver, never as 
executable SQL syntax — even if someone submits the exact same 
injection payload, it simply becomes a literal (non-matching) email 
value instead of altering the query's logic.

## What This Added To My Understanding
Seeing the real vulnerable code, and then the real fix, connected 
something I'd only described conceptually in every previous writeup's 
"Fix" section. Parameterized queries aren't just an abstract best 
practice  this is exactly what they look like in a real, production-
style codebase, and exactly why they work: separating query structure 
from user supplied data at the driver level, rather than trying to 
sanitize or escape input manually.


