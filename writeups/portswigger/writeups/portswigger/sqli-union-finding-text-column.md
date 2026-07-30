# PortSwigger Lab: SQL Injection UNION Attack - Finding a Column Containing Text

## Summary
Building on the previous lab's confirmed column count, this lab 
required identifying which specific column in the query is compatible 
with string/text data a necessary step before extracting text-based 
data (like usernames or passwords) in a full UNION attack.

## Vulnerability
Same injection point as previous labs: the product category filter 
reflects query results in the response and doesn't sanitize user input.

## Steps to Reproduce
1. Using the column count confirmed in the previous lab, attempt a 
   UNION SELECT with a test string value in the first column position:
   `' UNION SELECT '4I5bnq', NULL--`
   This returned an error, indicating the first column is not 
   compatible with string data (likely a numeric or other non-text type)
2. Move the test string to the second column position instead:
   `' UNION SELECT NULL, '4I5bnq'--`
   This succeeded, and the value `4I5bnq` appeared in the application's 
   response, confirming the second column accepts string data

## Impact
Identifying which column accepts string data is essential for any 
further UNION-based extraction attack, since data like usernames, 
passwords, and emails are stored as strings. Without this step, an 
attacker wouldn't know where in the query to place extracted data for 
it to display correctly.

## Fix
Use parameterized queries to prevent SQL injection at the source, and 
avoid exposing raw database errors to users, which reveal information 
about the underlying table structure and data types.

## What I Learned
This lab reinforced that UNION attacks aren't just about matching 
column *count* each column also has a specific data type, and values 
must match position by position. By testing the string in different 
column positions one at a time, I could isolate exactly which column 
was compatible with text. This makes sense now as the next logical step 
after counting columns: first match the shape (count), then match the 
type (compatibility) — both matter before real data extraction becomes 
possible.
