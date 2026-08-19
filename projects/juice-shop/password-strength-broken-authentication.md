# OWASP Juice Shop: Password Strength (Broken Authentication)

## Summary
The administrator account is protected by a weak, guessable password. 
Using Burp Suite's Intruder tool to automate a brute-force attack 
against the login endpoint, I was able to log in as 
`admin@juice-sh.op` without ever changing the password or using SQL 
injection simply by trying a short list of common weak passwords.

## Discovery Process
The challenge hint confirmed three valid approaches: manual guessing, 
attacking a harvested password hash, or a generic brute-force attack 
using a password list. I went with the brute-force approach using 
Burp Suite:

1. Intercepted a login request in Burp with a placeholder password
2. Sent the request to Intruder
3. Set the attack position on the `password` field only, keeping the 
   `email` field fixed to `admin@juice-sh.op`
4. Loaded a small list of common weak passwords (`admin123`, 
   `password`, `admin`, `123456`, `letmein`, `qwerty`, etc.)
5. Ran the attack

My first attempt failed entirely every request returned a `401` 
with an identical response length across all payloads. Rather than 
assuming none of the passwords were correct, I inspected the raw 
request body in Burp's Inspector and found the actual bug: a trailing 
space had been accidentally included in the email field during setup 
(`"admin@juice-sh.op "` instead of `"admin@juice-sh.op"`). This meant 
every single attempt was failing on the *email*, not the password
none of the guesses had actually been tested properly.

After correcting the email field and re-running the attack, the 
results changed immediately: the `admin123` payload returned a `200` 
status with a response length of `1170`, clearly distinct from every 
other `401`/`413` row. The response body contained a valid 
authentication token, confirming a successful login.

## Vulnerability
The administrator account's password (`admin123`) is a common, 
easily-guessable value with no enforced complexity requirements 
behind it. Combined with no apparent rate-limiting or lockout that 
stopped repeated automated login attempts from succeeding, this 
allowed a full account takeover via a small, generic brute-force 
attempt no advanced technique required.

## Steps to Reproduce
1. Intercept a login request to `/rest/user/login` in Burp Suite
2. Send the request to Intruder, marking only the `password` field as 
   the attack position (keep `email` fixed to `admin@juice-sh.op`)
3. Load a small list of common weak passwords as the payload set
4. Start the attack
5. Identify the request returning `200` instead of `401` (response 
   length also differs significantly from failed attempts)
6. Confirm the response body contains a valid authentication token
7. Log in manually on the actual login page using the discovered 
   credentials to confirm account access

## Impact
A weak administrator password with no apparent brute-force protection 
means the highest-privilege account in the application can be 
compromised using nothing more than a short list of common passwords 
and freely available tooling. In a real-world deployment, this would 
grant an attacker full administrative access — user data, order data, 
and any admin only functionality from outside the network, with no 
specialized exploit needed.

## Fix
- Enforce strong password requirements (minimum length, complexity) 
  for all accounts, especially privileged ones
- Implement account lockout or rate limiting on the login endpoint 
  after a threshold of failed attempts, to make brute-force attacks 
  impractical
- Consider requiring multi-factor authentication for administrative 
  accounts specifically, so a compromised password alone isn't 
  sufficient for full account access

## What I Learned
Beyond the core lesson that weak passwords plus unrestricted login 
attempts equal an easily brute-forceable account this challenge was 
a good reminder to actually inspect the raw request when results don't 
match expectations, rather than assuming the payload list itself is 
wrong. My first full attack run returned uniform failures across every 
password I tried, and it would have been easy to conclude none of my 
guesses were right and move on to a bigger wordlist. Instead, checking 
the actual request body revealed a single stray space in the email 
field was silently invalidating every attempt. It's a small thing, but 
it reinforced that when a brute-force or fuzzing attempt produces 
suspiciously uniform results, the fixed part of the request deserves 
as much scrutiny as the part being varied.


## Root Cause: Password Strength (Find It / Fix It)

## Find It
The vulnerability lives in the Sequelize model definition for the 
`User` entity, specifically in the password field's setter:

```javascript
User.init(
  password: {
    type: DataTypes.STRING,
    set (clearTextPassword: string) {
      this.setDataValue('password', security.hash(clearTextPassword))
    }
  },
```

The password is hashed and stored immediately, with no validation of 
any kind performed first no minimum length check, no complexity 
requirement, and no check against a list of known common/breached 
passwords. Whatever a user submits gets hashed and saved as-is, which 
is exactly what allowed `admin123` to be a valid password in the 
first place.

## Fix It
The accepted fix adds two validation checks before the password is 
allowed to be stored:

```javascript
set (clearTextPassword: string) {
  validatePasswordHasAtLeastTenChar(clearTextPassword)
  validatePasswordIsNotInTopOneMillionCommonPasswordsList(clearTextPassword)
  this.setDataValue('password', security.hash(clearTextPassword))
}
```

This enforces a minimum password length and rejects passwords that 
appear on a list of the top one million most common passwords — 
directly closing the gap that allowed a weak password like 
`admin123` to exist on a privileged account.

## What This Added To My Understanding
This challenge and the live brute-force exploitation above are really 
the same root vulnerability, observed at two different layers. In the 
live version, I saw the *symptom*: a weak admin password that a small, 
generic wordlist cracked in minutes. Here, in the source code, I found 
the actual *cause*: there was never any validation logic preventing a 
weak password from being set in the first place. Server-side password 
policy isn't enforced by convention or developer discipline it has 
to be an explicit check in the code path that handles credential 
storage, otherwise nothing stops an account (especially a 
high-privilege one like admin) from ending up with a trivially 
guessable password.
