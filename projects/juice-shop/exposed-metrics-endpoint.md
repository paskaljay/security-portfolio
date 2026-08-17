# OWASP Juice Shop: Exposed Metrics Endpoint (Observability Failure)

## Summary
The application exposes an unauthenticated `/metrics` endpoint 
intended for scraping by Prometheus, a popular open-source monitoring 
system. This endpoint reveals internal application usage data to 
anyone who requests it, with no authentication required.

## Discovery Process
The challenge hint mentioned "usage data scraped by a popular 
monitoring system," which pointed directly to Prometheus, since 
`/metrics` is its universal standard convention for applications to 
expose scrapeable data. Navigating directly to:
`http://localhost:3000/metrics`
immediately returned internal application metrics in plain text.

## Vulnerability
This is an Observability Failure / Sensitive Data Exposure issue — 
monitoring endpoints are meant to be scraped by internal, trusted 
monitoring infrastructure, not exposed publicly. Since there's no 
authentication on this route, anyone can view internal operational 
data about the application.

## Steps to Reproduce
1. Navigate directly to `http://localhost:3000/metrics`
2. Observe internal application metrics returned as plain text, with 
   no login or authorization required

## Impact
Exposed metrics endpoints can leak sensitive operational details — 
request patterns, error rates, internal route names, potentially even 
usernames or resource identifiers depending on what's tracked — giving 
an attacker reconnaissance information about the application's 
internals, usage patterns, and potential weak points, all without any 
authentication barrier.

## Fix
Monitoring endpoints like `/metrics` should be restricted to internal 
network access only (not publicly routable), or protected behind 
authentication specifically for the monitoring system's credentials, 
ensuring only trusted infrastructure can scrape this data.

## What I Learned
This was a quick but useful reminder that common framework/tooling 
conventions (like Prometheus's standard `/metrics` path) are worth 
knowing, since they're an easy first guess when hunting for exposed 
endpoints on any target using similar infrastructure. It also 
reinforced a theme I've now seen across several Juice Shop challenges: 
functionality meant for internal or trusted use only (admin routes, 
monitoring endpoints, internal directories) is only as safe as the 
access control actually placed on it — being "not meant for public 
use" is not the same as being protected.

## Root Cause: The Actual Vulnerable Code (Find It / Fix It Challenge)

The vulnerable route definition:

​```javascript
app.get('/metrics', utils.asyncHandler(metrics.serveMetrics()))
​```

This registers the `/metrics` endpoint with no authentication or 
authorization middleware in the chain, meaning any request — 
authenticated or not — gets served the internal metrics data directly.

## The Fix

​```javascript
app.get('/metrics', security.isAdmin(), utils.asyncHandler(metrics.serveMetrics()))
​```

The fix inserts `security.isAdmin()` as middleware between the route 
path and the handler. Express middleware runs sequentially, so this 
check executes first: if the requester isn't an authenticated admin, 
the request is rejected before `metrics.serveMetrics()` ever runs, 
meaning the actual data is never exposed to unauthorized requests.

## What This Added To My Understanding
This is a cleaner, more complete fix than the FTP directory 
vulnerability I found earlier rather than just removing a feature 
(like disabling directory listing) or relying on the endpoint being 
"unlisted," this properly restricts *access* itself, admin-only, 
regardless of whether someone knows the exact URL. This reinforced 
that the strongest fix for an exposed endpoint is almost always a real 
authorization check in the request path, not just reducing 
discoverability. It's a good reference pattern for evaluating other 
findings going forward: does the fix actually gate *access*, or does 
it just make the vulnerability harder to stumble upon by accident?
