# OWASP Juice Shop: Weird Crypto (Cryptographic Issues)

## Summary
Juice Shop uses MD5 for password hashing — A cryptographic algorithm 
long considered broken and unsuitable for password storage. This 
challenge required identifying and formally reporting that weakness 
through the application's Customer Feedback form.

## Discovery Process
This built directly on the earlier Password Hash Leak finding, where 
the admin account's password hash was retrieved:
`0192023a7bbd73250516f069df18b500`

This is a 32-character hexadecimal string — the characteristic length 
and format of an MD5 hash output. MD5 is well-documented as 
cryptographically broken: it's fast to compute (making brute-force 
attacks practical), vulnerable to collision attacks, and has been 
considered unsuitable for password hashing for well over a decade, 
in favor of purpose-built slow hashing algorithms like bcrypt, scrypt, 
or Argon2.

## Vulnerability
Using MD5 for password hashing means that even if an attacker only 
obtains the hash (not the plaintext password, as demonstrated in the 
Password Hash Leak finding), that hash can be cracked relatively 
quickly using modern hardware or precomputed rainbow tables, 
especially for common or weak passwords.

## Steps to Reproduce
1. Identify the hashing algorithm in use by examining a leaked password 
   hash's format and length (32 hex characters, consistent with MD5)
2. Navigate to the Customer Feedback / Contact form
3. Submit feedback identifying MD5 as an insecure algorithm the 
   application should not be using for password hashing
4. Solve the CAPTCHA and submit

## Impact
Combined with the earlier Password Hash Leak finding, this represents 
a realistic attack chain: an attacker who obtains a password hash 
through the leaky API endpoint could then crack it relatively easily 
due to the weak hashing algorithm, ultimately recovering the actual 
plaintext password. Weak hashing significantly amplifies the impact of 
any hash leaking vulnerability elsewhere in the application.

## Fix
Replace MD5 with a modern, purpose built password hashing algorithm 
such as bcrypt, scrypt, or Argon2. These algorithms are deliberately 
slow and computationally expensive, specifically to resist brute-force 
and rainbow table attacks, unlike general-purpose fast hash functions 
like MD5.

## What I Learned
This challenge connected directly back to an earlier finding, showing 
how vulnerabilities compound: the Password Hash Leak exposed the hash, 
and this challenge revealed that even if that leak were fixed, the 
underlying hashing algorithm itself is a separate, additional weakness. 
This reinforced that a thorough security review looks at multiple 
layers not just whether sensitive data is protected from exposure, 
but also whether the protection mechanism itself (in this case, the 
hashing algorithm) is actually sound. Recognizing a hash format by its 
length and characteristics is also a practical, transferable skill 
worth remembering for real-world assessments.
