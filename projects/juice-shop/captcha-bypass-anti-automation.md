# CAPTCHA Bypass (Broken Anti-Automation) — OWASP Juice Shop

## Summary
The Juice Shop feedback submission endpoint (`POST /api/Feedbacks/`) uses a
CAPTCHA to prevent automated/bulk submissions. However, the CAPTCHA answer
is never invalidated after a successful use. The same `captchaId` and
`captcha` value pair can be replayed indefinitely, allowing an attacker to
script mass submissions without ever solving a new CAPTCHA. This defeats
the entire purpose of the anti-automation control.

## Vulnerability
**Type:** Broken Anti-Automation / CAPTCHA Bypass
**Location:** `POST /api/Feedbacks/`
**Root cause:** The server validates the `captchaId` + `captcha` pair on
submission, but does not mark that CAPTCHA as "used" or expire it afterward.
This means a single solved CAPTCHA becomes a permanently reusable bypass
token, rather than a one-time proof of human interaction.

## Steps to Reproduce
1. Submit one legitimate feedback entry through the UI, solving the CAPTCHA
   normally. Capture this request in Burp Suite.
   - Example body included a solved pair: `captchaId: 2`, `captcha: "11"`.
2. In Burp Repeater, resend the exact same captured request (same
   `captchaId`/`captcha`) multiple times.
   - Result: every replay returned `201 Created` — confirming the CAPTCHA
     answer was never invalidated after first use.
3. Send the request to Burp Intruder.
   - **Positions:** mark only the `comment` field as the variable position,
     keeping `captchaId` and `captcha` fixed.
   - **Payloads:** Numbers payload type, 1 → 15, step 1 (generates 15 unique
     comment values so each request is a distinct feedback entry).
   - **Attack type:** Sniper.
4. Start the attack.
5. Result: all 16 requests (1 baseline + 15 payloads) returned `201 Created`,
   each responding in well under 50ms — completing the full batch in a
   fraction of the required 20-second window.
6. Confirmed on the scoreboard that the "CAPTCHA Bypass" challenge was
   marked solved.

## Impact
An attacker can script unlimited feedback (or any other CAPTCHA-protected
action) submissions, enabling:
- Spam / content flooding of user-facing feedback systems
- Automated abuse of any other endpoint reusing the same CAPTCHA pattern
- Defeating rate-limiting/anti-bot protections entirely, which can be a
  stepping stone to larger automated attacks (credential stuffing, fake
  account creation, etc. if the same CAPTCHA implementation is reused
  elsewhere)

This maps to **OWASP API Security Top 10 – API4:2023: Unrestricted Resource
Consumption**, and historically to the OWASP Automated Threats to Web
Applications category **OAT-018: Solving CAPTCHAs anywhere but the intended
flow / CAPTCHA bypass**.

## Fix
- Invalidate the CAPTCHA (`captchaId`) server-side immediately after its
  first successful use — a used CAPTCHA should never validate again.
- Tie each CAPTCHA to a single session/request and expire it after a short
  time window regardless of use.
- Add server-side rate limiting on the feedback endpoint independent of the
  CAPTCHA, so even a bypassed CAPTCHA can't enable unlimited submissions.
- Consider server-side behavioral detection (e.g., flagging near-identical
  rapid-fire requests from the same session/IP).

## What I Learned
- A CAPTCHA is only as strong as its *lifecycle management* — the hard part
  isn't generating the challenge, it's making sure the answer can't be
  reused.
- Burp Intruder's Sniper attack type is perfect for this: keep the exploited
  parameters fixed and vary only the field needed to make each request look
  like a "new" legitimate action.
- Confirmed firsthand how a manual Repeater test (2 replays) is a fast way
  to validate a hypothesis before scaling up to Intruder for full
  automation.
