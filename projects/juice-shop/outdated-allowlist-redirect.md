# OWASP Juice Shop: Outdated Allowlist (Unvalidated Redirects)

## Summary
The application's `/redirect` feature validates destination URLs 
against an allowlist before permitting the redirect. However, that 
allowlist still contains old cryptocurrency donation addresses that 
are no longer promoted or linked anywhere in the current UI, meaning 
these outdated entries remain valid redirect targets indefinitely, 
even though there's no legitimate reason for them to still be 
accepted.

## Discovery Process
Testing `http://localhost:3000/redirect?to=` with an empty or 
arbitrary URL returned a `406 Unrecognized target URL for redirect` 
error, confirming an allowlist check exists server-side. Since Juice 
Shop is open source, I looked into the actual project source code 
rather than reverse-engineering the minified bundle, and found the 
allowlist defined in `insecurity.js`:

​```javascript
const redirectAllowlist = new Set([
  'https://github.com/bkimminich/juice-shop',
  'https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm',
  'https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW',
  'https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6',
  ...
])
​```

The Bitcoin, Dash, and Ether donation addresses were confirmed to still 
be present in this list, despite no longer being linked or promoted 
anywhere in the current application UI.

## Vulnerability
This falls under Unvalidated Redirects — specifically, an allowlist 
that isn't properly maintained over time. Even though the redirect 
mechanism itself correctly restricts destinations to an approved list 
(good practice in principle), the list contains stale entries that 
should have been removed when the corresponding feature (crypto 
donations) was discontinued.

## Steps to Reproduce
1. Navigate to:
   `http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm`
2. The redirect succeeds, since this URL still matches an entry in the 
   allowlist, despite not being linked anywhere in the current 
   application

## Impact
While an allowlist-based redirect is a legitimate security control, 
failing to keep it updated as features change creates a false sense of 
security — old, no-longer-relevant entries remain exploitable 
indefinitely. In a real-world scenario, this pattern could be more 
severe if an old allowlisted domain were ever repurposed, expired, or 
compromised (e.g., an old marketing domain that lapses and gets bought 
by an attacker), since the application would still treat it as trusted.

## Fix
Allowlists should be reviewed and pruned whenever related features or 
integrations are removed, not left to accumulate stale entries 
indefinitely. Redirect allowlists specifically should be treated as 
living configuration tied to active application features, not a 
one-time setup.

## What I Learned
This challenge highlighted that having a security control in place 
(an allowlist) isn't sufficient on its own — it also needs ongoing 
maintenance to stay effective. I also learned a practical research 
technique: since Juice Shop is open source, checking the actual GitHub 
repository for relevant source files (like `insecurity.js`) was far 
more efficient than trying to manually parse minified/bundled 
JavaScript in DevTools. This is a useful reminder that when a target 
application's source is publicly available, going directly to it can 
save significant time compared to purely black-box reconnaissance.

## Root Cause: The Actual Vulnerable Code (Find It / Fix It Challenge)

Juice Shop's Find It / Fix It challenge confirmed the exact source 
code behind this vulnerability:

​```javascript
export const redirectAllowlist = new Set([
  'https://github.com/juice-shop/juice-shop',
  'https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm',
  'https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW',
  'https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6',
  'http://shop.spreadshirt.com/juiceshop',
  'http://shop.spreadshirt.de/juiceshop',
  'https://www.stickeryou.com/products/owasp-juice-shop/794',
  'http://leanpub.com/juice-shop'
])
​```

The vulnerable lines are the three cryptocurrency donation address 
entries — these remained in the allowlist despite the crypto donation 
feature no longer existing anywhere in the current application UI.

## The Fix

The fix removes the outdated cryptocurrency entries entirely, leaving 
only the URLs that correspond to features still actively promoted in 
the application (GitHub repo, merchandise shops, book link).

## A Related, Separate Vulnerability Worth Noting
While reviewing this code, I also noticed the allowlist check itself 
has a separate weakness, unrelated to which URLs are on the list:

​```javascript
export const isRedirectAllowed = (url: string) => {
  let allowed = false
  for (const allowedUrl of redirectAllowlist) {
    allowed = allowed || url.includes(allowedUrl)
  }
  return allowed
}
​```

This uses `url.includes(allowedUrl)`, which checks whether an approved 
URL appears *anywhere within* the submitted URL, rather than requiring 
an exact match or verified domain start. This means a crafted URL like 
`https://github.com/bkimminich/juice-shop.evil.com` could potentially 
pass this check, since it contains the trusted string as a substring, 
even though the actual destination domain is entirely different. This 
appears to be a distinct vulnerability from the outdated allowlist 
issue, likely covered by a separate challenge, and highlights that 
allowlist *content* and allowlist *validation logic* are two separate 
things that both need to be correct.

## What This Added To My Understanding
This confirmed that the fix for a stale allowlist is straightforward — 
just remove entries tied to discontinued features. But it also 
surfaced a more subtle, related issue in the same file: even a 
perfectly maintained allowlist can still be bypassed if the comparison 
logic itself is weak. This taught me to look at both the *data* 
(what's on the list) and the *logic* (how the list is checked) 
separately when reviewing access control or validation code, since a 
vulnerability can exist in either one independently of the other.
