# OWASP Juice Shop: Admin Section (Broken Access Control)

## Summary
The application's administration section is reachable simply by 
navigating directly to its route no admin privileges are actually 
enforced. While logged in as a regular, non-privileged user, I was 
able to load the admin page and view its full contents, including 
data that should only be visible to an administrator.

## Discovery Process
The admin section has no visible link anywhere in the UI for a normal 
user it's hidden from navigation, but hidden is not the same as 
protected. Based on the same pattern I'd seen with the exposed 
`/metrics` endpoint and the `/redirect` destination validation issue 
(features that exist and function but aren't gated by actual access 
control), I guessed that an `/administration` route likely existed 
client-side and tried navigating directly to it:

/#/administration

## Vulnerability
The application relies on the absence of a UI link to keep the admin 
section "hidden," rather than enforcing role-based access control on 
the route or its underlying data. Since the route and its data are 
client-side, anyone who knows or guesses the URL logged in as any 
user, with no special role can load it directly.

## Steps to Reproduce
1. Log in as a normal, non-admin user
2. Navigate directly to `/#/administration` in the browser's URL bar
3. The full administration page loads, with no permission check or 
   redirect, despite the current user having no admin privileges

## Impact
This is a serious access control failure. Any authenticated user 
and potentially an unauthenticated one, depending on how the 
underlying data is fetched can view administrative data intended 
to be restricted, such as user account details. In a real-world 
application, this could expose sensitive user data, business logic, 
or administrative controls (like user/content deletion) to anyone 
capable of guessing a URL.

## Fix
Access control must be enforced server-side, not assumed from the 
absence of a UI link. The application should:
- Check the authenticated user's role on both the route (client-side 
  guard) and, more importantly, on every API call the admin page 
  makes, rejecting requests from non-admin users with a 403
- Never treat "not linked in the UI" as equivalent to "protected" 
  this is security through obscurity, not real access control

## What I Learned
This is now the third finding that shares the same root cause: 
`/metrics`, `/redirect`, and now the admin section all rely on 
obscurity or client-side assumptions instead of real server-side 
enforcement. It's a strong pattern in how this application handles 
access control hiding a feature from the UI is treated as if it 
were the same as securing it, when in reality any route or endpoint 
that isn't explicitly checked against the user's role/permissions on 
the backend is fully accessible to anyone who finds it.


# Root Cause: Admin Section (Find It / Fix It)

## Find It
The vulnerability lives in the Angular routing configuration:

```typescript
{
  path: 'administration',
  component: AdministrationComponent,
  canActivate: [AdminGuard]
},
```

At first glance this looks protected — there's a `canActivate: [AdminGuard]` 
guard attached to the route. However, this is exactly what I 
demonstrated in the live exploitation above: as a logged-in, 
non-admin user, I was still able to reach `/#/administration` and 
view its full contents despite this guard supposedly being in place.

## Fix It
The accepted fix doesn't patch or strengthen `AdminGuard` — it removes 
the route from this application's routing table entirely:

```typescript
/* TODO: Externalize admin functions into separate application
   that is only accessible inside corporate network.
*/
// {
//   path: 'administration',
//   component: AdministrationComponent,
//   canActivate: [AdminGuard]
// },
```

## Why This Is the Fix, Not Just a Patch
This surprised me at first the guard already existed, so why not 
just fix the guard? The answer connects directly to what I observed 
in the live version of this challenge: a client-side route guard is 
enforced in code that ships to and runs in the user's own browser. 
Even if the guard is implemented correctly, it's still a control 
living inside an environment the user fully controls, which makes it 
inherently fragile  one bug, one bypass, or one gap in how the 
underlying API is secured, and the guard means nothing (which is 
exactly what happened when I accessed the section as a regular user).

The real fix treats this as an architecture problem, not a bug: admin 
functionality should not live inside the same public-facing 
application as the storefront at all. Externalizing it into a 
separate application one that's only reachable from inside the 
corporate network removes the sensitive functionality from public 
exposure entirely, rather than trying to gate access to it with a 
check that runs on the client.

## What This Added To My Understanding
This is a good example of the difference between fixing a symptom and 
fixing a root cause. Patching or hardening `AdminGuard` would only 
address this specific bypass; it wouldn't change the fact that 
sensitive administrative functionality is shipped as part of a 
public-facing client application in the first place. This reinforces 
the same theme from `/metrics`, `/redirect`, and the live version of 
this exact challenge: security controls that live entirely in 
client-side code (guards, hidden routes, disabled UI elements) are 
fundamentally different from controls enforced by architecture or the 
server — and this challenge's fix is one of the few that explicitly 
acknowledges that by removing the client-side surface instead of 
trying to secure it.
