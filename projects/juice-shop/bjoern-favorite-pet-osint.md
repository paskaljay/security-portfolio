# OWASP Juice Shop: Bjoern's Favorite Pet (Broken Authentication / OSINT)

## Summary
This challenge required resetting the real Juice Shop creator's 
(Bjoern Kimminich) actual OWASP account password using the genuine 
answer to his security question discoverable because he mentioned 
his pet's name publicly during a recorded talk, effectively 
self doxxing the answer to his own security question.

## Discovery Process
1. Accessed the Administration panel (from an earlier admin-access 
   challenge) to find Bjoern's registered email addresses within the 
   application, identifying `bjoern@owasp.org` as one of them
2. Used the Forgot Password mechanism with this email, revealing the 
   security question: "Name of your favorite pet?"
3. Researched publicly available information about Bjoern Kimminich, 
   finding that he mentioned his cat's name, "Zaya," during a public 
   conference talk that was recorded and made available online
4. Used "Zaya" as the security answer to successfully reset the 
   password

## Vulnerability
This is a Broken Authentication issue rooted in a real-world OSINT 
risk: security questions relying on personal information (a pet's 
name) can be discovered through publicly available content the 
account holder themselves shared, even inadvertently, such as in a 
recorded presentation.

## Steps to Reproduce
1. Obtain the target's registered email via the Administration panel
2. Navigate to the Forgot Password page and enter that email
3. Research the target's public presence (talks, social media, 
   interviews) for information matching the security question category
4. Submit the discovered answer to reset the password

## Impact
This demonstrates that even a system's own creator can be vulnerable 
to the same class of weakness their application is built to teach 
about publicly shared personal information, even shared 
unintentionally or in passing during a talk, can directly compromise 
account security when used as a security question answer. This is a 
genuinely realistic risk: public speakers, streamers, and anyone with 
an online presence routinely share small personal details that could 
map directly onto common security question categories (pets, first 
car, hometown, etc.).

## Fix
Avoid using security questions based on personal facts that could 
plausibly be shared publicly, even in seemingly unrelated contexts 
like conference talks or casual social media posts. Prefer stronger 
account recovery mechanisms, such as email based reset links or 
multi factor authentication, that don't depend on secrecy of a 
memorable personal fact.

## What I Learned
This challenge, alongside MC SafeSearch and the geo stalking 
challenges, reinforced that OSINT based account compromise is a 
genuinely distinct and important skill from technical exploitation
and importantly, it isn't limited to fictional in game personas. Here, 
the "victim" is the actual real world creator of the application, and 
the vulnerability stems from something he plausibly said once, in an 
unrelated context, years ago. This is a valuable reminder for my own 
future conduct: casually mentioning personal details in public 
recordings, talks, or social media can create durable, searchable 
security risks that persist indefinitely online.
