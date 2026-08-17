# OWASP Juice Shop: Confidential Document via Unprotected File Directory

## Summary
The application exposes an `/ftp` directory containing multiple 
sensitive files with no authentication or authorization checks 
whatsoever. Simply navigating directly to the directory URL reveals a 
full file listing, and any file within it — including confidential 
business documents — can be accessed by anyone, logged in or not.

## Discovery Process
The challenge hint suggested "analyzing and tampering with links that 
deliver a file directly" and noted the target file "is not protected 
in any way." This pointed toward looking for downloadable file links 
elsewhere on the site (such as invoices) to infer a file path pattern, 
which led to trying the root FTP-style directory directly:
`http://localhost:3000/ftp`

This returned a full, unauthenticated directory listing containing 
several files, including:
- `acquisitions.md`
- `legal.md`
- `incident-support.kdbx` (a KeePass password database file — likely 
  tied to a separate challenge)
- `announcement_encrypted.md`
- Various `.bak` backup files

## Vulnerability
This is a Sensitive Data Exposure / Security Misconfiguration issue — 
the directory and its contents have no access control at all. There's 
no login requirement, no permission check, and no indication anywhere 
in the normal UI that this directory exists; it's purely accessible by 
guessing or discovering the URL.

## Steps to Reproduce
1. Navigate directly to `http://localhost:3000/ftp`
2. Observe the full, unrestricted file listing
3. Click on a document with a confidentiality-suggesting name (e.g. 
   `legal.md`)
4. The file opens/downloads as plain text, fully readable, with no 
   authentication prompt at any point

## Impact
Any sensitive business document placed in this directory — legal 
files, acquisition details, backup files potentially containing 
credentials or configuration data — is accessible to literally anyone 
who finds or guesses the URL. This is a serious exposure risk, since 
`.bak` backup files in particular often contain leftover sensitive 
data (old configs, credentials, source code) that developers forget 
to remove or protect.

## Fix
Directories serving files directly should never be publicly listable 
or accessible without authentication. Sensitive files should be moved 
outside the publicly served web root entirely, or access should 
require proper authentication and authorization checks before any file 
is served, rather than relying on the directory simply not being 
linked anywhere in the visible UI (the same false "security through 
obscurity" issue seen in the Score Board challenge).

## What I Learned
This challenge reinforced a pattern I first saw in the Score Board 
challenge: assuming something is safe just because it isn't linked in 
the visible navigation is not real security. Here, an entire directory 
of genuinely sensitive-sounding files (legal documents, acquisition 
details, backup files) was sitting completely open, discoverable just 
by guessing a common directory name like `/ftp`. This is a realistic 
example of a very common real-world misconfiguration — backup files 
and internal documents accidentally left in a publicly accessible web 
directory — and a good reminder to always check for exposed 
directories and backup file extensions (`.bak`, `.old`, `.orig`) when 
testing a real target.


## Root Cause: The Actual Vulnerable Code (Find It / Fix It Challenge)

After discovering and exploiting the `/ftp` directory listing 
manually, Juice Shop's Find It / Fix It challenge revealed the exact 
server-side routing code responsible:

​```javascript
app.use('/ftp', serveIndexMiddleware, serveIndex('ftp', { icons: true }))
app.use('/ftp(?!/quarantine)/:file', servePublicFiles())
app.use('/ftp/quarantine/:file', serveQuarantineFiles())
​```

The vulnerable line is the first one — `serveIndex('ftp', ...)` is 
Express.js middleware that automatically generates a browsable file 
listing for the `/ftp` directory, with no authentication middleware 
placed before it in the chain. This is exactly what produced the file 
listing page showing `acquisitions.md`, `legal.md`, and other files 
when navigating directly to `/ftp` in the browser.

## The Fix

The fix removes the `serveIndex` line entirely:

​```javascript
app.use('/ftp(?!/quarantine)/:file', servePublicFiles())
app.use('/ftp/quarantine/:file', serveQuarantineFiles())
​```

This disables the auto-generated directory listing, so the folder's 
contents are no longer browsable at a glance. 

## An Important Nuance
Notably, the individual file-serving routes (`servePublicFiles()`, 
`serveQuarantineFiles()`) remain unchanged — meaning if an attacker 
already knows or guesses an exact filename, the file may still be 
directly accessible. This fix reduces risk significantly by removing 
the convenient "here's everything in one list" discovery method, but 
doesn't fully eliminate exposure on its own; a more complete fix would 
also add authentication/authorization checks to the file-serving 
routes themselves, not just the listing.

## What This Added To My Understanding
Seeing the real code confirmed that this vulnerability came from a 
single middleware call with no access control gate in front of it a 
very common and easy-to-make oversight, since `serveIndex` is a 
legitimate, commonly-used Express package for exactly this purpose 
(directory browsing), just applied here without appropriate 
protection. It also taught me an important distinction between a 
*complete* fix and a *partial* one: removing directory listing stops 
casual discovery, but doesn't guarantee the underlying files are 
actually protected if their names become known through other means 
(like this very writeup, ironically). I also noticed this same file 
contains identical `serveIndex` patterns for `/encryptionkeys` and 
`/support/logs` directories — likely separate, still-undiscovered 
challenges using the exact same vulnerability pattern.
