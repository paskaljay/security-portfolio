# OWASP Juice Shop: Deprecated Interface (Security Misconfiguration)

## Summary
The Customer Complaint form's file upload feature still accepts XML 
files server side, despite the visible file picker restricting 
selection to PDF and ZIP only. This is a leftover remnant of a 
deprecated B2B feature that once allowed customers to upload XML 
invoices/orders when the feature was removed from normal use, the 
underlying server side support for XML uploads was never actually 
disabled.

## Discovery Process
Initial attempts to find a hidden API endpoint (guessing URL patterns 
like `/b2b/v1/orders` based on the current `/b2b/v2` structure visible 
in the Swagger API docs) were unsuccessful those routes didn't exist 
and just fell through to the app's default catch-all route. Research 
into the actual challenge revealed the real deprecated interface was a 
file upload feature, not a separate API endpoint. The complaint form's 
file picker only allowed selecting `.pdf` or `.zip` files by default, 
but this restriction exists purely in the frontend file input's 
`accept` attribute, not as an actual server-side validation.

## Vulnerability
The B2B order upload feature (originally accepting XML invoices) was 
deprecated and removed from the visible UI, but the server endpoint 
handling file uploads was never updated to reject XML files — only the 
frontend's file picker dialog was restricted, which is trivially 
bypassable.

## Steps to Reproduce
1. Navigate to the Customer Complaint page
2. Fill in required fields (name, message)
3. When selecting a file for upload, bypass the frontend's file type 
   restriction (either via the file picker's format dropdown, if 
   available, or by allowing all file types)
4. Select and upload a simple XML file
5. Submit the complaint — the upload succeeds despite XML no longer 
   being an officially supported/promoted file type

## Impact
This demonstrates that removing a feature from the visible UI does not 
mean the underlying functionality is actually gone the deprecated 
B2B XML upload path remained fully functional and reachable by anyone 
willing to bypass a client side file picker restriction. In more 
severe cases, this exact pattern (an old, unmonitored XML upload path 
still being active) is often the entry point for more serious 
vulnerabilities like XXE (XML External Entity) injection, since 
developers and security reviews may have stopped scrutinizing a 
feature they believe was already removed.

## Fix
When deprecating a feature, both the frontend AND backend code paths 
need to be removed or properly disabled — restricting only the visible 
UI (like a file picker's accepted extensions) provides no real 
security boundary, since it's fully controllable by the client.

## What I Learned
This challenge was a strong, concrete example of a theme I've now seen 
repeatedly throughout this project: frontend restrictions are 
convenience features for normal users, never security controls. I also 
learned a valuable research lesson here my first instinct (guessing 
API URL patterns based on the current version's structure) was a 
reasonable approach that didn't happen to be correct this time, and 
switching to researching the actual challenge context (what feature 
was being deprecated, and how) got me to the right answer faster than 
continued guessing would have. This is a good reminder that when 
black-box guessing stalls, stepping back to understand the broader 
feature history or context can reveal the actual vulnerable path more 
directly.
