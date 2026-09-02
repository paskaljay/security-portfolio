# OWASP Juice Shop: Visual Geo Stalking (Sensitive Data Exposure / OSINT)

## Summary
Similar to the Meta Geo Stalking challenge, this required resetting a 
user's password by determining the answer to their security question — 
but this time using visual details directly visible within an uploaded 
photo, rather than hidden EXIF metadata.

## Discovery Process
1. Identified Emma's photo on the Photo Wall by process of elimination: 
   most other uploaded photos were attributed to Bjoern or John, 
   leaving one captioned "My old workplace... (© E=ma²)" — a stylized 
   reference to "Emma"  as the relevant image
2. Checked the security question via Forgot Password using 
   `emma@juice-sh.op`, which asked for something resembling a business 
   or employer name
3. Downloaded and closely zoomed into the photo (a building exterior), 
   revealing a small, easy to miss sign or banner reading "ITsec" in 
   one of the windows the actual answer to the security question

## Vulnerability
This is the same underlying issue as the Meta Geo Stalking finding: a 
user's security question answer was discoverable through content they 
themselves publicly shared, this time through visible details in the 
image itself rather than hidden metadata. Security questions relying 
on information that can be independently discovered through public 
content are inherently weak.

## Steps to Reproduce
1. Locate the target user's relevant photo on the Photo Wall
2. Download and closely inspect the image for visible identifying 
   details (signage, landmarks, text)
3. Check the security question via the Forgot Password mechanism
4. Use the visually discovered detail as the answer to reset the 
   password

## Impact
Even without any hidden metadata, a photo's actual visible content can 
leak information sufficient to answer common security questions (former 
employer, workplace, etc.). This demonstrates that sensitive data 
exposure isn't limited to obviously "technical" leaks like metadata or 
API responses visual content itself can be a leak vector, especially 
when combined with predictable security question categories.

## Fix
Avoid using security questions with answers that could plausibly be 
discovered through a user's public photos, social media, or other 
shared content such as former employers, schools, or pet names. 
Prefer stronger account recovery methods (e.g. email-based verification 
links, multi-factor authentication) over knowledge-based security 
questions entirely.

## What I Learned
This challenge, paired with Meta Geo Stalking, reinforced that OSINT
based vulnerabilities can come from multiple angles within the same 
piece of content hidden metadata in one case, visible details in 
another. It also required a different skill than most of my other 
findings: careful visual inspection and zooming rather than technical 
tooling. This is a good reminder that real world reconnaissance often 
benefits from simply looking closely at content a target has shared, 
not just running automated tools against it.
