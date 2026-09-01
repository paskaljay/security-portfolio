# OWASP Juice Shop: Password Hash Leak (Sensitive Data Exposure)

## Summary
The `/rest/user/whoami` endpoint accepts a `fields` query parameter 
that lets the client specify exactly which user fields to return. 
While intended for legitimate fields like `email`, it also allows 
requesting `password`, returning the currently logged-in user's 
password hash directly in the API response — data that should never 
be exposed to the client at all, even in hashed form.

## Discovery Process
Initial attempts to access user data directly (e.g. 
`/api/Users/1`) failed with a 401 Unauthorized error when visited 
directly in the browser address bar, since that bypasses the 
JavaScript-managed Authorization token normally attached to requests. 
After confirming the correct endpoint pattern 
(`/rest/user/whoami`) and that it accepts a `fields` parameter (seen 
in a captured request using `?fields=email`), I extended the parameter 
to request the password field as well:
`http://localhost:3000/rest/user/whoami?fields=email,password`

## Vulnerability
​```
GET /rest/user/whoami?fields=email,password
​```
Response:
​```json
{"user":{"email":"admin@juice-sh.op","password":"0192023a7bbd73250516f069df18b500"}}
​```
The endpoint performs no restriction on which fields can be requested 
via this generic field-selection mechanism, allowing a client to 
request sensitive fields like `password` that were never intended to 
be exposed through this API, even in hashed form.

## Steps to Reproduce
1. Log in normally to obtain an authenticated session
2. Navigate directly to:
   `http://localhost:3000/rest/user/whoami?fields=email,password`
3. Observe the password hash returned directly in the JSON response

## Impact
While the password is stored hashed (not plaintext), exposing the 
hash at all is a serious issue hashes can potentially be cracked via 
rainbow tables or brute force, especially for common/weak passwords. 
This also reveals implementation details (the hashing algorithm's 
output format) that could aid an attacker in planning further attacks. 
Generic, unrestricted field-selection APIs like this are a known 
anti-pattern precisely because they can't anticipate every sensitive 
field a client might request.

## Fix
Explicitly define an allowlist of fields that are permitted to be 
requested through this endpoint, rejecting or ignoring any request for 
fields outside that list — such as `password`, password hashes, or 
other internal/sensitive data that should never be client-accessible 
regardless of the requesting user's identity.

## What I Learned
This challenge reinforced the risk of overly generic, flexible APIs — 
allowing clients to specify arbitrary fields via a query parameter is 
convenient for legitimate use cases, but without an explicit allowlist, 
it becomes an open door to any field that exists on the underlying data 
model, sensitive or not. This is a great real-world example of why API 
design needs to default to restrictive (only exposing what's explicitly 
needed) rather than permissive (exposing everything unless specifically 
blocked) — a lesson that generalizes well beyond this specific endpoint.
