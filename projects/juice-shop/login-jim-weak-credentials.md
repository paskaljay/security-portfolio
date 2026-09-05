# OWASP Juice Shop: Login Jim (Sensitive Data Exposure)

## Summary
Unlike the Login Admin challenge (solved via SQL injection), this 
challenge was solved using Jim's actual, valid credentials 
`jim@juice-sh.op` / `ncc-1701`. This password is a known reference 
(the registry number of the USS Enterprise from Star Trek) associated 
with this specific Juice Shop character, discoverable through research 
into the application's fictional user personas, similar to the earlier 
MC SafeSearch OSINT finding.

## Discovery Process
Rather than exploiting a code level vulnerability, this required 
recognizing or researching the themed credential associated with 
Jim's character within the Juice Shop universe.

## Steps to Reproduce
1. Navigate to the login page
2. Email: `jim@juice-sh.op`
3. Password: `ncc-1701`
4. Log in successfully with no injection or bypass technique required

## Impact
This demonstrates the same category of risk as the MC SafeSearch 
finding: predictable or thematically guessable passwords tied to a 
user's known persona or interests can be discovered without any 
technical exploitation at all. In a real application, this underscores 
why password strength policies and guidance against personally 
meaningful (and therefore guessable) passwords matter.

## Fix
Enforce strong password policies that reject easily-guessable or 
culturally referenced passwords, and avoid any association between a 
user's public persona/interests and their actual credentials.

## What I Learned
This was a good reminder that not every "login" challenge in this 
project involved a technical exploit sometimes user research and 
recognizing thematic patterns (much like the MC SafeSearch challenge) 
is the actual vulnerability being tested, reinforcing that credential 
security has both a technical dimension (injection, hashing) and a 
human dimension (password choice, predictability) that both matter in 
real-world security assessments.
