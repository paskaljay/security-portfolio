# OWASP Juice Shop: Exposed Credentials (Sensitive Data Exposure)

## Summary
A developer left a valid set of login credentials for a testing 
account hardcoded directly in the application's client-side 
JavaScript. Anyone opening the browser's DevTools can retrieve these 
credentials and log in as a real, working account no exploitation 
required beyond reading shipped source code.

## Discovery Process
Using Chrome DevTools, I opened the Sources panel and used the global 
search (Cmd+Opt+F) to search across all loaded JavaScript bundles. An 
initial search for the word "password" returned hundreds of matches, 
most of which were unrelated UI strings (labels, validators, error 
messages). I narrowed the search to the domain pattern `@juice-sh.op`, 
which this app conventionally uses for its test/fake accounts, and 
found an email/password pair sitting together in the bundled source:

email: testing@juice-sh.op
password: IamUsedForTesting

## Vulnerability
Hardcoding credentials even for an "unused" testing account — into 
client-side code means the credentials ship to every user's browser 
as plain, readable text. Client-side JavaScript is never a secure 
place to store secrets, since anyone can inspect it with tools built 
into every browser.

## Steps to Reproduce
1. Open DevTools → Sources panel
2. Use global search (Cmd+Opt+F) across all loaded files
3. Search for the app's test-account email domain (`@juice-sh.op`) 
   rather than generic terms like "password," which returns excessive 
   noise
4. Locate the email/password pair in the bundled JS
5. Log out of the current session
6. Log in using the discovered credentials at the standard login page
7. Login succeeds, confirming the credentials are live and valid

## Impact
Even though this particular account is labeled as "unused" for 
testing, the underlying issue is serious: any credential shipped in 
client-side code is exposed to every visitor of the site, not just 
authenticated users. In a real-world application, this same mistake 
could expose credentials for an account with real privileges 
admin access, internal tooling, or a service account  turning a 
"harmless" testing artifact into a full account compromise.

## Fix
Never hardcode credentials of any kind including test accounts  
into client-side code. Test credentials should be injected at build 
time via environment variables that are stripped from production 
bundles, or the testing account should be removed entirely before 
deployment. Additionally, any account not intended for production use 
should be disabled or deleted rather than left dormant with valid 
credentials.

## What I Learned
This challenge highlighted how "temporary" or "just for testing" code 
often ships to production by accident, and how forgiving generic 
search terms are of that kind of leftover cruft searching narrowly 
for a recognizable pattern (the email domain) cut through the noise 
far faster than searching for a common word like "password." It's a 
good reminder that source code review and secret scanning shouldn't 
just look for the word "password," but for realistic credential-shaped 
patterns.
