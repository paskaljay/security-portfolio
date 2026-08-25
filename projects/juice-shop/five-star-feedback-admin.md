# OWASP Juice Shop: Five-Star Feedback (Broken Access Control)

## Summary
The application includes an Administration panel that allows deleting 
customer feedback entries. Accessing this panel and removing all 
5-star rated feedback entries confirms the admin section is properly 
gated behind authentication, but also demonstrates the kind of 
privileged data-management access an admin account grants.

## Steps to Reproduce
1. Log in as the `administrator` user (using the SQL injection login 
   bypass technique from an earlier challenge: `' OR 1=1--` in the 
   email field)
2. Navigate to the Administration panel:
   `http://localhost:3000/#/administration`
3. Locate the customer feedback table, which lists all submitted 
   feedback along with their star ratings
4. Delete every feedback entry rated 5 stars using the delete button 
   provided in the admin interface

## Impact
While this specific action (deleting feedback) is intended admin 
functionality rather than a vulnerability on its own, reaching this 
point relies on the same login bypass vulnerability documented in an 
earlier finding once administrator access is obtained through that 
SQL injection flaw, the attacker gains full access to this 
administrative data management interface, including the ability to 
delete customer submitted content at will.

## Fix
This challenge reinforces the importance of the earlier login bypass 
fix (parameterized queries) the Administration panel itself appears 
properly restricted to authenticated admin sessions, but that 
restriction is only as strong as the authentication mechanism 
protecting it. If login can be bypassed, every admin-only feature 
downstream of it becomes accessible too.

## What I Learned
This challenge was a good reminder that a single vulnerability early 
in an attack chain (the SQL injection login bypass) can cascade into 
access over many other features that are otherwise properly access
controlled. The Administration panel itself isn't broken it 
correctly requires an admin session but because the *authentication* 
step itself was compromised much earlier in my testing, everything 
downstream of it became reachable. This connects back to a lesson from 
my very first Juice Shop challenge and reinforces why authentication 
vulnerabilities are often treated as especially high severity: they 
don't just expose one feature, they can expose everything gated behind 
that login.
