# OWASP Juice Shop: Admin Registration (Improper Input Validation)

## Summary
Juice Shop uses a library (Finale) that automatically generates full 
REST API CRUD endpoints from database models, including user 
registration. While sensitive fields like `password` and `totpSecret` 
were explicitly excluded from being settable via this auto generated 
API, the `role` field was not excluded, allowing a new account to be 
registered directly with `role: "admin"`, bypassing the intended 
default of `customer`.

## Discovery Process
After successfully bypassing the frontend registration form's fields 
via Burp Suite (adding a `role` field not present in the normal form), 
I registered a new account with `"role":"admin"` included in the 
request body and successfully logged in with full administrator 
privileges.

## Vulnerability
​```javascript
const autoModels = [
  { name: 'User', exclude: ['password', 'totpSecret'], model: UserModel },
  ...
]
​```
The `exclude` array only protects `password` and `totpSecret` from 
being client settable during registration — `role` remains fully 
exposed as a writable field through the auto generated API, with no 
server-side enforcement enforcing the intended default of `customer`.

## Steps to Reproduce
1. Intercept a normal registration request using Burp Suite
2. Add a `role` field to the JSON body with value `"admin"`
3. Send the modified request
4. Log in with the newly created account's credentials
5. Confirm administrator level access (e.g., access to the 
   Administration panel)

## Impact
This is a critical vulnerability anyone can create a fully 
privileged administrator account with zero authentication or prior 
access required, simply by registering normally and adding one extra 
field to the request. This completely undermines the application's 
entire privilege/role system.

## Fix
​```javascript
resource.create.send.before((req, res, context) => {
  WalletModel.create({ UserId: context.instance.id })...
  context.instance.role = 'customer'
  return context.continue
})
​```
The fix forcibly overwrites the `role` field to always be `'customer'` 
immediately before the new user record is saved, regardless of what 
value the client submitted in the request. This ensures the role can 
never be set to anything else through the registration endpoint, no 
matter what the client sends.

## What I Learned
This was a great example of how auto-generated CRUD APIs (a common 
convenience pattern in many frameworks) can silently expose more than 
intended the developers correctly thought to exclude `password` and 
`totpSecret`, but overlooked that `role` was equally sensitive and 
needed the same protection. This reinforces a broader lesson: when 
using any tool that automatically generates API surface from a data 
model, every field needs to be deliberately reviewed for whether it 
should be client-writable, not just the "obviously sensitive" ones 
like passwords. A single overlooked field in an exclude list can 
undermine an entire privilege system.
