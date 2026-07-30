# PortSwigger Lab: SQL Injection UNION Attack - Determining Column Count

## Summary
The product category filter is vulnerable to SQL injection, and the 
query results are reflected in the application's response. This allows 
determining the exact number of columns returned by the original query 
using a UNION-based technique a required first step before extracting 
data from other tables in later attacks.

## Vulnerability
The category filter builds a query that reflects its results back to 
the user, and the `category` parameter isn't sanitized, allowing 
injected SQL to run alongside the original query.

## Steps to Reproduce
1. Confirm the injection point is vulnerable by submitting a single 
   quote (`'`) in the category parameter, which caused a database error
2. Attempt a UNION attack starting with one NULL value:
   `' UNION SELECT NULL--`
   This still returned an error, indicating the original query returns 
   more than one column
3. Increase to two NULL values:
   `' UNION SELECT NULL,NULL--`
   This succeeded without error, confirming the original query returns 
   exactly two columns
4. NULL is used instead of real values at this stage because NULL is 
   compatible with virtually any data type, avoiding type-mismatch 
   errors while the exact column count and types are still unknown

## Impact
While this step alone doesn't extract data, it's the foundation for a 
full UNION-based data extraction attack. Once the column count is known, 
an attacker can begin substituting real queries into those positions 
to pull data from arbitrary tables (e.g. usernames and passwords from 
a users table).

## Fix
Use parameterized queries to prevent user input from being interpreted 
as executable SQL, and avoid reflecting raw query structure/errors back 
to users, which can leak information useful for constructing these 
attacks.

## What I Learned
This lab taught me the systematic approach to a UNION attack: 
incrementally testing NULL values until the query stops erroring, which 
reveals the exact column count needed for any injected SELECT statement 
to match the original query's structure. I also learned why NULL 
specifically is used for this step it sidesteps data type mismatches 
that would otherwise cause errors even when the column count itself is 
correct. I don't yet have a fully intuitive grasp of UNION as a concept, 
but working through this step-by-step made it click more than reading 
about it did, and I expect the next labs (using this column count to 
actually extract data) will reinforce it further.
