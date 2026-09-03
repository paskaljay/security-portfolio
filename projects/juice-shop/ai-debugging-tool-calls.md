# OWASP Juice Shop: AI Debugging (Broken Access Control)

## Summary
The chatbot's debugging feature — which displays internal tool calls 
and shows how the AI interacts with backend functions — is intended 
for administrators only. However, access control for this feature is 
implemented entirely client-side via a cookie, allowing any user 
(including non-admins) to enable it by simply setting that cookie 
value manually.

## Discovery Process
Using DevTools' source search, I located the relevant frontend code 
and found the cookie being read:
`this.showToolCalls.set(this.cookieService.get("show_tool_calls") === "true")`

This confirmed the actual cookie name (`show_tool_calls`, distinct 
from the similarly-named `showToolCalls` JavaScript variable) and that 
its value is checked purely client-side, with no corresponding 
server-side authorization check for whether the requesting user is 
actually an admin.

## Vulnerability
Setting the `show_tool_calls` cookie to `true` manually via browser 
DevTools reveals internal tool call details in the chatbot's responses 
(e.g., `searchProducts{"query":"under $5 products"}`), regardless of 
the logged-in user's actual role. Notably, this only counts as solving 
the challenge when done as a non-admin user, since an admin naturally 
has legitimate access to this view already.

## Steps to Reproduce
1. Log in as a regular (non-admin) user
2. Open DevTools → Application → Cookies
3. Manually add a cookie named `show_tool_calls` with value `true`
4. Interact with the chatbot using a query likely to trigger a backend 
   tool call (e.g., asking about products matching specific criteria)
5. Observe internal tool call details appearing in the chat response, 
   despite not having admin privileges

## Impact
Exposing internal tool call details to regular users leaks 
implementation information about the chatbot's backend capabilities 
and how it interacts with the application's data layer information 
that could help an attacker understand the system's internals for 
further exploitation (e.g., crafting more effective prompt injection 
attacks by knowing exactly which tools/functions are available to the 
AI).

## Fix
Implement server side authorization checks for the debugging feature, 
verifying the requesting user's actual role from their authenticated 
session rather than relying on a client controlled cookie value that 
any user can freely modify.

## What I Learned
This challenge reinforced a pattern I've now seen many times 
throughout this project: client-side-only access control is not real 
access control, since cookies, local storage, and any browser-stored 
state are fully within the user's control. I also want to note the 
significant infrastructure debugging this required beforehand — 
getting a local Ollama LLM properly connected to a Dockerized Juice 
Shop instance involved working through Docker networking 
(`host.docker.internal`), node-config's file loading precedence 
(`default.yml` vs `local.yml` vs `NODE_ENV`-based files), and correct 
YAML nesting (`application.chatBot.llmApiUrl`, not a top-level key). 
This was a good reminder that a meaningful portion of real security 
testing work involves environment setup and infrastructure 
troubleshooting, not just the vulnerability exploitation itself.
