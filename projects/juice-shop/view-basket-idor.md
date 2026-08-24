# OWASP Juice Shop: View Basket (Broken Access Control / IDOR)

## Summary
The application stores the logged-in user's shopping basket ID (`bid`) 
directly inside their JWT authentication token, and the basket-fetching 
endpoint trusts whatever basket ID is requested without verifying it 
actually belongs to the authenticated user. This allows viewing any 
other user's shopping basket simply by changing the ID in the request 
URL.

## Discovery Process
While inspecting an authenticated request in Burp Suite, I noticed the 
JWT stored in the `Authorization: Bearer` header decodes to reveal a 
`"bid": 1` field the user's basket ID, embedded directly in a 
client controlled token. This matched the challenge hint about a 
"client-side association of user to basket." Locating the actual 
basket fetching request (`GET /rest/basket/:id`), I changed the ID in 
the URL to a different number and resent the request.

## Vulnerability
​```
GET /rest/basket/1
​```
Changing this to a different basket ID (e.g. `/rest/basket/2`) 
successfully returned another user's basket contents, confirming the 
endpoint performs no ownership check it only verifies that a valid 
basket ID was provided, not that the requesting user actually owns it.

## Steps to Reproduce
1. Log in normally and capture an authenticated request in Burp Suite
2. Locate the basket-fetching request, `GET /rest/basket/:id`
3. Change the ID in the URL to a different, arbitrary number
4. Resend the request
5. Observe that another user's basket contents are returned, despite 
   the request being authenticated as a completely different user

## Impact
Any authenticated user can view any other user's shopping basket 
contents simply by guessing or incrementing basket IDs, exposing 
private purchase intent data across all users on the platform. This is 
the same underlying vulnerability class as the earlier Forged Review 
finding missing ownership verification just affecting a different 
feature (baskets instead of reviews).

## Fix
The basket-fetching endpoint should verify that the requested basket 
ID actually belongs to the currently authenticated user (matching 
against their session/token identity) before returning any data, 
rather than trusting the ID alone as sufficient authorization.

## What I Learned
This challenge reinforced a pattern I'm now seeing repeatedly across 
this application: authentication (proving who you are) and 
authorization (verifying what you're allowed to access) are treated as 
the same thing in several places, when they need to be checked 
separately. I also learned something new about JWTs specifically 
since they're just Base64-encoded (not encrypted), any data embedded 
in them, like this basket ID, is fully readable by the client. Storing 
an authorization relevant value like a basket ID inside a token that 
the client can read (and in this case, effectively also request 
different values for via the URL) is a risky pattern if the backend 
doesn't independently re-verify ownership on every request.
