# PortSwigger Lab: Visible Error-Based SQL Injection

## Summary
This lab contained a SQL injection vulnerability in a tracking cookie, 
where query results weren't directly returned — but a deliberately 
caused type-conversion error exposed the actual data value inside the 
error message text itself, allowing direct extraction of the 
administrator's username and password.

## Vulnerability
The `TrackingId` cookie value is inserted directly into a SQL query 
without sanitization. The database is PostgreSQL, confirmed by using 
`LIMIT` syntax successfully (Oracle would require a different clause 
like `ROWNUM` or `FETCH FIRST`).

## Steps to Reproduce
1. Confirmed the injection point using a single quote in the tracking 
   cookie, which caused an error
2. Confirmed comment syntax worked using `'--` to close out the rest 
   of the original query safely
3. Constructed a payload that forces a type-conversion error, 
   deliberately trying to cast a text value into an integer:
   `' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
4. The resulting error message included the actual value that failed 
   conversion, directly in the response body:
   `ERROR: invalid input syntax for type integer: "administrator"`
   This confirmed the technique works and leaked the first username
5. Repeated the same technique targeting the password column instead:
   `' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`
   This leaked the corresponding password directly in the error 
   message:
   `ERROR: invalid input syntax for type integer: "ewakc28x2vpu70c5x1qs"`
6. Logged in as `administrator` using the leaked password

## Impact
Error-based SQL injection allows direct data extraction through error 
messages alone, without needing visible query output (like UNION-based 
attacks) or slow, indirect inference (like blind boolean-based attacks). 
This makes it one of the fastest and most dangerous SQLi variants when 
verbose database errors are exposed to the end user, since a single 
crafted request can leak an entire piece of sensitive data at once.

## Fix
Use parameterized queries to eliminate SQL injection at the source. 
Critically, never expose raw database error messages to end users — 
generic, non-descriptive error pages prevent this exact technique, 
since the attacker relies entirely on the database's verbose error 
output being visible in the response.

## What I Learned
This lab showed me a middle ground between UNION-based extraction 
(fast but requires visible query output) and blind boolean-based 
extraction (works with no visible output, but painfully slow, one 
character at a time). Error-based injection gets the speed of UNION 
attacks even when the application doesn't display query results 
directly — as long as verbose database errors leak into the response. 
I also learned to identify database type from behavior rather than 
assuming: `LIMIT` working (instead of the `dual`/`TO_CHAR` syntax from 
my earlier Oracle-based lab) told me this was PostgreSQL, which meant 
adjusting my approach based on real feedback rather than reusing the 
same syntax blindly.
