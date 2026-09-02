# OWASP Juice Shop: Security Policy (Miscellaneous / Good Practice)

## Summary
Applications following responsible security practices publish a 
security.txt file at a standardized location, describing how 
researchers should report vulnerabilities responsibly rather than 
disclosing them publicly or exploiting them maliciously. This 
challenge involved locating and reviewing that file for Juice Shop 
itself, reflecting the standard a "white-hat" hacker should follow 
before engaging in vulnerability testing.

## Discovery Process
Security.txt follows RFC 9116, a real internet standard specifying 
that this file should be located at:
`/.well-known/security.txt`
Navigating directly to this path on the running Juice Shop instance 
revealed the actual file.

## What Was Found
​```
Contact: mailto:donotreply@owasp-juice.shop
Encryption: https://keybase.io/bkimminich/pgp_keys.asc?fingerprint=...
Acknowledgements: /#/score-board
Preferred-languages: en, ar, az, bg, ...
Hiring: /#/jobs
Csaf: http://localhost:3000/.well-known/csaf/provider-metadata.json
Expires: Wed, 01 Sep 2027 19:45:30 GMT
​```

## Significance
This isn't a vulnerability at all it's the opposite: a demonstration 
of a genuinely good security practice. The file provides a clear, 
standardized way for security researchers to report findings 
responsibly (contact info, encryption key for sensitive reports, and 
even a CSAF — Common Security Advisory Framework — link for structured 
vulnerability disclosure data), rather than researchers having to guess 
how to responsibly disclose an issue or resorting to public disclosure.

## What I Learned
This challenge was a good reminder that responsible disclosure isn't 
just an abstract ethical concept — it's backed by real standards 
(RFC 9116) and conventions that real applications and companies 
implement in practice. As I continue toward real bug bounty hunting, 
checking for a `security.txt` file on any target should be one of the 
first things I do — it tells me exactly how the organization wants 
vulnerabilities reported, whether they have a bug bounty program, and 
sets the expectation for ethical engagement before I do anything else. 
This challenge fittingly reinforces the broader principle behind 
everything I've practiced on Juice Shop: testing and reporting 
vulnerabilities responsibly, on authorized targets, through proper 
channels.
