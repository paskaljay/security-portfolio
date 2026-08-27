# OWASP Juice Shop: Login MC SafeSearch (Sensitive Data Exposure / OSINT)

## Summary
This challenge required logging into a fictional user's account using 
their real, original credentials discovered not through any 
technical exploit, but through open-source intelligence (OSINT) 
gathering. The account belongs to "MC SafeSearch," a fictional rapper 
character created by the Juice Shop team, who reveals his own password 
in a real YouTube music video.

## Discovery Process
1. Found the user's email address via the Administration panel 
   (accessible from an earlier admin-login challenge), which lists all 
   registered users: `mc.safesearch@juice-sh.op`
2. Searched online for "MC SafeSearch," which led to a real YouTube 
   music video titled "Protect Ya' Passwordz" an easter egg created 
   specifically for this challenge
3. In the song's lyrics, the character reveals his password is based 
   on his dog's name, "Mr. Noodles," with the letter O's replaced by 
   zeros, yielding: `Mr. N00dles`
4. Logged in normally using these credentials, with no injection or 
   bypass technique involved

## Vulnerability
This is a Sensitive Data Exposure issue: the user's actual password 
was leaked publicly, outside the application entirely, through a 
piece of external content (a music video) associated with that 
persona. It highlights that credential security isn't just about the 
application's own codeinformation leaked anywhere publicly and 
associated with a user can be discovered and used against them, 
regardless of how well the login form itself is secured.

## Steps to Reproduce
1. Find the target user's email via the Administration panel
2. Search publicly for information linked to that user's persona
3. Locate the leaked password reference (in this case, a YouTube 
   video)
4. Log in normally using the discovered email and password

## Impact
Even a login form with no technical vulnerabilities at all can be 
compromised if the user's actual credentials are discoverable through 
public information social media posts, leaked videos, forum posts, 
or any content that hints at or directly states password patterns. 
This is a real world risk category, not just a CTF gimmick: many 
actual account compromises happen through OSINT and social engineering 
rather than technical exploitation of the application itself.

## Fix
This isn't something the application itself can technically prevent 
the fix here is user education: never reference real password patterns 
or specifics in any public content, and use unique, unguessable 
passwords rather than personally meaningful (and therefore discoverable 
or guessable) information like a pet's name.

## What I Learned
This challenge was a good reminder that not every vulnerability 
category is a code level bug sometimes the "attack surface" is the 
user's own public footprint rather than the application itself. It 
also reinforced why security awareness training for end users matters 
alongside technical application security: no amount of secure coding 
prevents someone from voluntarily leaking their own password pattern 
in a public video, blog post, or social media comment. This was also a 
nice change of pace from purely technical exploitation, requiring 
research and lateral thinking instead of payload crafting.
