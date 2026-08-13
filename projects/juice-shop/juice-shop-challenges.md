## OWASP Juice Shop: Hidden Score Board Page (Security Misconfiguration)

## Summary
The application's Angular routing configuration is shipped in full to 
the client browser, including routes for pages never linked in the 
visible UI — such as the Score Board admin/debug page. Since the 
entire route list is readable client-side, "hiding" a page by simply 
not linking to it provides no real protection.

## Vulnerability
This is not SQL injection — it falls under Security Misconfiguration / 
Sensitive Information Exposure. Client-side routing files are always 
fully visible to anyone inspecting the app's source, regardless of 
whether a route is linked in the visible navigation menu.

## Steps to Reproduce
1. Inspected the app's Angular routing file (visible in browser 
   dev tools / bundled JS)
2. Found a route entry for the Score Board page that isn't linked 
   anywhere in the normal UI
3. Navigated directly to that path, gaining access to the hidden page

## Impact
Any "hidden" functionality relying purely on the absence of a UI link 
is not actually protected — since client-side code is always fully 
visible, an attacker can simply read the routing configuration to 
discover every page the application supports, bypassing intended 
obscurity entirely.

## Fix
Interestingly, Juice Shop's own guidance here is to leave the routing 
code unchanged — removing or obscuring the route wouldn't fix the 
underlying issue and could break legitimate access to the page. The 
real fix isn't a code change to the routing file at all, but proper 
access control: sensitive pages should require authentication/ 
authorization checks server-side, regardless of whether the route is 
publicly listed or not.

## What I Learned
This challenge taught me that not every vulnerability has a clean, 
isolated code fix — sometimes "hiding" something client-side is a 
false sense of security, and the real fix is architectural (proper 
access control) rather than something you patch in one line. It also 
reinforced that not everything is SQL injection — recognizing when a 
vulnerability belongs to a completely different category (security 
misconfiguration here, not injection) is just as important as knowing 
injection techniques themselves.
