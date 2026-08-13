# OWASP Juice Shop: IDOR in Forged Review (Broken Access Control)

## Summary
The product review update endpoint allowed any authenticated user to 
edit *any* review, not just their own, because the database query only 
checked the review's ID and never verified the requester actually 
owned that review. This is a classic Insecure Direct Object Reference 
(IDOR) vulnerability.

## Vulnerability
​```javascript
const user = security.authenticatedUsers.from(req)
db.reviewsCollection.update(
  { _id: req.body.id },
  { $set: { message: req.body.message } },
  { multi: true }
)
​```
The code fetches the currently authenticated `user`, but never actually 
uses it anywhere in the update query — the query only filters by 
`_id`, meaning any logged-in user could submit any review's ID along 
with a new message, and the server would update it regardless of 
actual ownership.

This uses MongoDB (a NoSQL database), identifiable by the `$set` 
operator, the `_id` default field name, and the JavaScript 
object-based query syntax — a different query language than the SQL 
databases used in the PortSwigger labs, but the same underlying 
missing-authorization concept.

## Steps to Reproduce
1. Identified the vulnerable update query, noting that `user` was 
   fetched but never referenced in the actual database filter
2. Confirmed the fix required adding an ownership check to the query 
   filter, changing:
   `{ _id: req.body.id }`
   to:
   `{ _id: req.body.id, author: user.data.email }`
3. This ensures the update only succeeds if both the ID matches AND 
   the review's author matches the currently authenticated user

## Impact
Without the ownership check, any authenticated user could overwrite 
any other user's review content simply by knowing or guessing its ID — 
a form of unauthorized data tampering affecting every review on the 
platform, not just the attacker's own.

## Fix
Add an ownership condition to the database query filter, requiring 
that the requesting user's identity matches the review's recorded 
author before allowing any update to proceed. Fetching the 
authenticated user is only useful if that identity is actually enforced 
in the query — simply knowing who made the request isn't a security 
control unless it's checked against the data being modified.

## What I Learned
This was a genuinely valuable exercise in reading real backend code 
for a missing check rather than a wrong technique unlike SQL 
injection, where malicious syntax causes the problem, IDOR here was 
caused by an *absence* of logic, not the presence of anything harmful. 
Spotting that `user` was fetched but never used was the key clue, and 
the fix was small (adding one field to the query filter) but 
conceptually important: authentication (knowing who someone is) and 
authorization (checking what they're allowed to do) are two separate 
things, and this bug happened because only the first was implemented, 
not the second. I also got my first real exposure to MongoDB/NoSQL 
query syntax, which is meaningfully different from the SQL databases 
I've worked with so far, though the security principle (validate 
ownership, not just existence) transfers directly across both.
