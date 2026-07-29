# PortSwigger Lab: SQL Injection - Retrieving Hidden Data

## Summary
The application filters products by category using a SQL query that 
directly concatenates user input, making it vulnerable to SQL injection. 
This allows bypassing the `released = 1` condition to view unreleased products.

## Vulnerability
The category filter builds a query like:
​```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
​```
Since the category value comes from user input without sanitization, 
an attacker can inject SQL logic instead of a plain category name.

## Steps to Reproduce
1. Navigate to the product category page and intercept the request (e.g. with Burp Suite)
2. Identify the `category` parameter in the request
3. Modify the parameter to break out of the intended query logic and 
   neutralize the `released = 1` condition — for example, by injecting 
   a payload that comments out the rest of the query or forces the 
   condition to always be true
4. Send the modified request and observe that unreleased products 
   now appear in the response

## Impact
This vulnerability allows an attacker to bypass application logic 
entirely — in this case viewing products that should be hidden from 
regular users. In more severe real-world cases, similar injection 
points can allow reading/modifying/deleting arbitrary database data, 
or even full database compromise.

## Fix
Use parameterized queries (prepared statements) instead of directly 
concatenating user input into SQL queries. This ensures user input is 
always treated as data, never executable SQL code.

## What I Learned
I first tested the category parameter with a single quote to check for 
SQL injection — this caused a database error, confirming the input 
wasn't being sanitized or parameterized. From there, I injected a 
payload combining an always-true condition with a comment sequence to 
neutralize the rest of the query. This showed me how just two syntax 
elements — a boolean tautology and a comment marker — can completely 
override an application's intended access control logic. It also 
reinforced why testing for errors first is a fast, low-risk way to 
confirm a vulnerability exists before committing to a full exploit.
