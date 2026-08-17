# OWASP Juice Shop: DOM-Based XSS in Product Reviews

## Summary
The product review submission field is vulnerable to DOM-based XSS. 
Unlike the customer feedback form (which properly sanitizes input), 
the review field inserts user input directly into the page via 
client-side JavaScript without escaping, allowing injected HTML/JS to 
execute in the browser.

## Discovery Process
This challenge's hint specifically warned that it's "almost 
indistinguishable from reflected XSS unless you look under the hood" 
meaning the vulnerable field wasn't obvious at first glance, and 
several payloads and fields needed to be tested before finding what 
actually worked:

1. First tried a basic HTML tag, `<h1><owasp>`, in the search bar 
   this didn't trigger anything, just resulted in a blank/broken 
   results page
2. Tried `<script>alert('xss')</script>` next — this also didn't 
   execute, likely because script tags inserted via certain DOM methods 
   (like `innerHTML`) are not executed by browsers as a security 
   measure, even when there's no server-side filtering
3. Tried `<iframe src="javascript:alert('xss')">` in the search bar  
   this successfully triggered an alert, confirming the search bar 
   itself has a DOM XSS vulnerability, though this did not register as 
   solving the "Forged Review" scoreboard challenge specifically, 
   indicating the search bar is a separate, distinct XSS challenge
4. Tried the Customer Feedback form's Comment field with a similar 
   payload — the form accepted submission after solving its CAPTCHA, 
   but no alert triggered, indicating this field properly sanitizes 
   input before rendering it
5. Used the same working iframe payload on the product review field 
   instead, which successfully triggered the alert and was confirmed 
   as solving this specific challenge

## Vulnerability
This is DOM-based XSS specifically (not reflected/server-based) — the 
review text is inserted into the page's HTML via client-side 
JavaScript without sanitization, meaning the malicious payload never 
needs to round-trip through the server to execute; the browser 
executes it directly when rendering the review.

## Steps to Reproduce
1. Navigate to a product page and open the review submission section
2. In the review text field, enter:
   `<iframe src="javascript:alert('xss')">`
3. Submit the review
4. The injected iframe executes its `javascript:` URL, triggering an 
   alert box — confirming arbitrary JavaScript execution

## Impact
An attacker could submit a malicious review that executes JavaScript 
in the browser of any user who later views that product's reviews — 
potentially stealing session tokens, redirecting users to phishing 
pages, or performing actions on their behalf without their knowledge.

## Fix
Sanitize and escape all user-submitted content before inserting it 
into the DOM, or use safe rendering methods (like `textContent` 
instead of `innerHTML`) that treat user input strictly as text, never 
as executable HTML/JavaScript.

## What I Learned
This challenge involved genuine trial and error across multiple 
payload types before finding one that worked — plain HTML tags and 
`<script>` tags both failed, while an `<iframe>` with a `javascript:` 
URL succeeded, which taught me that different injection techniques 
bypass different filtering mechanisms in different ways; there's no 
single universal XSS payload that works everywhere. I also learned 
that not every input field on a page shares the same sanitization 
behavior — the Customer Feedback form and the product review field 
looked like they might behave similarly, but only one was actually 
vulnerable. Additionally, triggering an alert doesn't automatically 
confirm you've solved a *specific* tracked challenge — the search bar 
and product review field turned out to be separate challenges 
entirely, despite both being exploitable with the same payload.

## Correction: The Exact Payload That Mattered

After further investigation, the scoreboard's specific hint for this 
challenge revealed the exact expected payload:

​```html
<iframe src="javascript:alert(`xss`)">
​```

Note the use of **backticks** (`` ` ``) around `xss`, rather than 
single quotes (`alert('xss')`), which is what I had been using 
throughout my earlier attempts. Both are valid JavaScript syntax for 
defining a string and both execute identically in the browser  my 
single-quote version did successfully trigger alert popups in both the 
search bar and product review field. However, the challenge's 
completion check appears to look for an exact string match rather than 
just confirming that *any* XSS payload executed, meaning the specific 
backtick syntax was required for the scoreboard to register the 
challenge as solved.

## Additional Lesson Learned
This taught me an important distinction between **confirming a 
vulnerability exists** (which my original payload did, successfully, 
multiple times) and **matching a specific automated success 
condition** (which required the exact payload syntax). In a real bug 
bounty context, proving the vulnerability exists — as I did with the 
single-quote version — would be sufficient for a valid report; exact 
payload matching only matters here because Juice Shop's challenge 
tracking system checks for a specific string. This is a useful 
reminder that CTF-style completion checks and real-world vulnerability 
validation aren't always the same thing.

## Root Cause: The Actual Vulnerable Code (Find It / Fix It Challenge)

After exploiting this blind (without seeing the code), Juice Shop's 
Find It / Fix It challenge revealed the actual vulnerable line:

​```javascript
this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam)
​```

Angular (the framework this app is built with) automatically sanitizes 
user input by default before rendering it — a strong built-in 
protection against XSS. `bypassSecurityTrustHtml()` is a special 
Angular function that explicitly disables this protection for a given 
value, telling the framework to trust and render it as raw HTML 
without sanitization. Here, it was applied directly to `queryParam`, 
raw user input taken straight from the URL's search query string, 
never validated or sanitized first.

## The Fix

​```javascript
this.searchValue = queryParam
​```

The fix simply removes the `bypassSecurityTrustHtml()` call entirely, 
assigning the raw query parameter directly. This allows Angular's 
default sanitization to apply normally, meaning any injected HTML or 
JavaScript in the search query gets safely escaped and rendered as 
plain text rather than executed.

## What This Added To My Understanding
This was a genuinely valuable root-cause discovery: I had already 
exploited this vulnerability without knowing why it existed, assuming 
it was simply a case of missing sanitization. Seeing the actual code 
revealed something more specific and honestly more common in 
real-world applications a developer *explicitly* disabling a modern 
framework's default XSS protection, likely for a legitimate seeming 
reason (perhaps wanting to allow some HTML formatting in search 
results) without properly scoping or validating that decision. This 
reinforced an important lesson: frameworks like Angular and React are 
often naturally resistant to XSS by default, and most real-world XSS 
in these frameworks comes from developers deliberately opting out of 
that protection, not from the framework itself failing.
