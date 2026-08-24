# OWASP Juice Shop: Empty User Registration (Improper Input Validation)

## Summary
The user registration endpoint accepts submissions with completely 
empty `email` and `password` fields when the frontend form is 
bypassed, despite both fields being marked as required in the visible 
UI. This is the same underlying pattern as the Repetitive Registration 
and Zero Stars findings validation exists only in the frontend, not 
enforced server side.

## Discovery Process
Using the same approach as previous frontend bypass findings, I 
intercepted the registration request via Burp Suite rather than 
relying on the form's required field validation, which prevents 
submission with empty inputs through the normal UI.

## Vulnerability
​```json
{
  "email": "",
  "password": "",
  "passwordRepeat": "",
  ...
}
​```
Submitting this directly to the registration API endpoint succeeds, 
confirming the backend never validates that `email` and `password` are 
actually non-empty that constraint exists only as a frontend "field 
required" check.

## Steps to Reproduce
1. Intercept a registration request using Burp Suite
2. Modify the request body so `email` and `password` are empty strings
3. Forward the modified request
4. Observe a successful account creation response despite the empty 
   required fields

## Impact
Allowing accounts with empty credentials undermines basic account 
integrity an account with no email means no way to contact or 
recover it, and an empty password is trivially exploitable, since 
anyone could potentially log into such an account with a blank 
password field. This could also pollute the user database with 
unusable or duplicate empty credential accounts.

## Fix
Enforce required field validation server side for `email` and 
`password`, rejecting any registration request where these fields are 
missing or empty, rather than relying solely on the frontend form's 
required field UI behavior.

## What I Learned
This is now the third distinct feature (registration passwords, 
feedback ratings, and now basic required fields) where I've found the 
exact same root cause: frontend-only validation with no server-side 
enforcement. At this point, this looks less like isolated oversights 
and more like a systemic pattern in how this application's forms were 
built validation was treated as a UX concern only, never duplicated 
as an actual security/data integrity control on the backend. This 
reinforces that when auditing a real application, finding one instance 
of frontend only validation is a strong signal to test *every* other 
form on the same application for the identical weakness, since the 
same development habit likely repeats across features.
