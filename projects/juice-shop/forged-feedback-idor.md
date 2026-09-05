# OWASP Juice Shop: Forged Feedback (Broken Access Control / IDOR)

## Summary
The customer feedback submission endpoint accepts a client supplied 
`UserId` field, allowing feedback to be posted under any user's 
identity  including the administrator regardless of who is actually 
logged in and submitting the request.

## Discovery Process
While intercepting a normal feedback submission in Burp Suite, I found 
the request body included a `UserId` field alongside the expected 
comment, rating, and CAPTCHA fields:
​```json
{
  "UserId": 2,
  "captchaId": 0,
  "captcha": "36",
  "comment": "...",
  "rating": 4
}
​```
Changing this value to `1` (the administrator's user ID) and 
forwarding the request successfully posted the feedback under the 
admin account instead of my own.

## Vulnerability
The feedback endpoint trusts the client-supplied `UserId` value 
directly, rather than deriving the submitting user's identity from 
their authenticated session (e.g., their JWT token). This is the same 
underlying pattern as the earlier Forged Review vulnerability the 
server has no check verifying that the `UserId` in the request 
actually matches the currently authenticated user.

## Steps to Reproduce
1. Log in as any regular user
2. Fill out the Customer Feedback form and solve the CAPTCHA
3. Intercept the submission request using Burp Suite
4. Change the `UserId` field in the request body to a different user's 
   ID (e.g., `1` for administrator)
5. Forward the modified request
6. Confirm via the Administration panel that the feedback now appears 
   attributed to the targeted user, not the actual submitter

## Impact
Any authenticated user can post feedback that appears to come from 
any other user, including administrators enabling impersonation, 
reputation damage, or planting misleading/malicious content under a 
victim's name without their knowledge or consent.

## Fix
The server should derive the submitting user's identity from their 
authenticated session (JWT token) rather than trusting a client-
supplied `UserId` field in the request body. Any `UserId` value sent 
by the client for this purpose should be ignored entirely in favor of 
the server verified identity.

## What I Learned
This is now the third distinct instance of the same underlying flaw 
across different features (Forged Review, View Basket via JWT, and now 
Forged Feedback) trusting a client supplied identifier instead of 
deriving identity from the authenticated session. Seeing this exact 
pattern repeat across three separate endpoints reinforces that it's 
likely a systemic habit in how this application handles user 
association generally, not isolated oversights. In real-world testing, 
finding this pattern once is a strong signal to check every other 
endpoint that accepts any kind of user/owner identifier for the same 
weakness.
