# OWASP Juice Shop: Zero Stars (Improper Input Validation)

## Summary
The customer feedback form's rating slider only allows selecting 
values from 1 to 5 stars through the visible UI. However, the backend 
API accepts a rating of 0 when submitted directly, bypassing the 
frontend's enforced minimum the same class of frontend-only 
validation issue seen in the Repetitive Registration challenge.

## Discovery Process
Using the same approach as the Repetitive Registration finding, I 
intercepted the feedback submission request using Burp Suite rather 
than relying on the UI's rating slider, which visually prevents 
selecting below 1 star.

## Vulnerability
The feedback submission endpoint doesn't validate that the `rating` 
field falls within the intended 1–5 range server-side it only relies 
on the frontend slider component to enforce this boundary.

## Steps to Reproduce
1. Fill out the Customer Feedback form normally, solve the CAPTCHA
2. Intercept the submission request using Burp Suite before it reaches 
   the server
3. Modify the `rating` field in the request body to `0`
4. Forward the modified request
5. The server accepts the submission successfully, despite the rating 
   falling outside the UI's intended 1–5 range

## Impact
Allowing out-of-range values to be stored can cause downstream issues 
beyond just displaying an odd rating average rating calculations, 
sorting/filtering logic, or any code assuming ratings always fall 
within 1–5 could behave unexpectedly or incorrectly when a 0 (or 
potentially negative) value is present in the data.

## Fix
Validate the `rating` field server-side, rejecting any submission 
where the value falls outside the intended 1–5 range, rather than 
relying solely on the frontend UI component to enforce this 
constraint.

## What I Learned
This challenge reinforced the exact same lesson from Repetitive 
Registration: UI components like sliders, dropdowns, or disabled 
buttons only restrict what a *normal user* can do through the browser 
they provide zero actual security or data integrity guarantee, since 
any request can be crafted and sent directly to the API, bypassing the 
UI entirely. Seeing this same pattern appear in two different features 
(registration and feedback) suggests it may be a broader habit in how 
this application's validation was implemented, rather than an isolated 
oversight.
