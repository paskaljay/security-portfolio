# OWASP Juice Shop: Meta Geo Stalking (Sensitive Data Exposure)

## Summary
A user (John) uploaded a photo to the Photo Wall without stripping its 
EXIF metadata, which included embedded GPS coordinates. Using those 
coordinates, I was able to identify the location shown in the photo 
and use it to correctly answer his account's security question, 
allowing a full password reset via the "Forgot Password" mechanism 
without any technical exploitation of the app itself.

## Discovery Process
1. Navigated to the Photo Wall and located a photo captioned to 
   indicate it belonged to John (hiking-themed image)
2. Downloaded the image and extracted its EXIF metadata using an 
   online EXIF viewer, revealing GPS coordinates: 
   `36°57'31.38" N, 84°20'53.58" W`
3. Plugged the coordinates into Google Maps, which placed the location 
   within Daniel Boone National Forest
4. Went to "Forgot Password" and entered John's email 
   (`john@juice-sh.op`) to reveal his security question, which related 
   to a park he enjoys hiking in
5. Submitted "Daniel Boone National Forest" as the answer, which was 
   accepted, allowing the password to be reset

## Vulnerability
This isn't a traditional application vulnerability in code  it's a 
demonstration of how personal metadata embedded in uploaded files, 
combined with weak/guessable security questions, can be chained 
together to fully compromise an account. The app allowed an image 
with unstripped EXIF/GPS data to be publicly uploaded and viewed, and 
paired that with a security question whose answer could be derived 
directly from that same data.

## Steps to Reproduce
1. Locate a target user's photo on the Photo Wall
2. Download the image and inspect its EXIF metadata for embedded GPS 
   coordinates
3. Look up those coordinates to determine the real-world location 
   depicted
4. Use "Forgot Password" with the target's email to view their 
   security question
5. Answer using the location information derived from the photo's 
   metadata
6. Complete the password reset and log in as the target user

## Impact
This chain shows how sensitive location data embedded in an innocuous 
photo upload can lead directly to full account takeover, entirely 
outside of any traditional exploit like SQL injection or XSS. In a 
real world context, this is a genuine and common attack path: users 
routinely upload photos without realizing GPS metadata is attached, 
and security questions based on personal/derivable facts (hometowns, 
favorite places, etc.) are inherently weak once any OSINT is possible.

## Fix
- Strip EXIF/GPS metadata from all user-uploaded images server-side 
  before storing or serving them publicly
- Avoid security questions altogether where possible, in favor of 
  stronger account recovery methods (e.g., email-based reset links, 
  multi-factor authentication)
- If security questions must be used, avoid ones whose answers can be 
  derived from publicly available or easily inferred personal 
  information

## What I Learned
This challenge was a good reminder that not every vulnerability lives 
in the application's code some come from how the app handles user 
data (unprocessed file uploads) combined with weak account-recovery 
design (security questions). It also reinforced that "security 
questions" as a concept are fundamentally weak in the age of social 
media and readily available metadata: anything a user has ever posted 
publicly, or any file they've uploaded without sanitization, can 
become an attack vector for answering a question that was assumed to 
be private knowledge only the account owner would have.
