# SQL Injection

## Summary
This section covers a SQL injection vulnerability in a web-based version of the gift card application implemented using Django. Unlike some of the other vulnerabilities, which targeted the original C-based implementation, this issue arises within the Python-based web interface and its HTML components.

To exploit this vulnerability, I created a gift card that injected a SQL query to retrieve admin credentials from the database.

**Note:** The original source files were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerable component was the gift card file upload functionality in `views.py`. It parsed user inputs without validating or sanitizing them, which made the application vulnerable to SQL injection.

## Vulnerability Details
The root cause of the vulnerability was that user-uploaded gift card files were not sanitized, and no prepared statements were used for the `signature` field. So whatever the user entered as the signature could be used to query the database and retrieve stored credentials.

```
Card.objects.raw('select id from ... where data = \'%s\'' % signature)
```
Because no prepared statements were used here, the `signature` field input is directly executed as part of the SQL query.

## Steps to Reproduce
To obtain the password for an admin user, I created a gift card which contained the following SQL query in the `signature` field.
```
"signature": "1' UNION select username||' '|| password from ... where username = 'admin"
```
This gift card file was then uploaded to the application and the exposed credentials were displayed on the UI.

## Impact
Exploiting this vulnerability allowed credentials to be stolen from the database. In a real-world scenario, this could be exploited further to carry out large-scale data breaches and account takeovers.

## Mitigation
To fix this vulnerability, I modified the gift card upload functionality to use prepared statements.
```
Card.objects.raw('select id from ... where data = %s', [signature])
```
Using prepared statements forces the database to treat user input as plain data instead of part of the SQL query. This prevents a user's `signature` field input from being executed, therefore protecting against SQL injection.

## Tools and Artifacts Used
- SQL queries (to craft the payload)
- Gift card file payloads (to trigger the injection)
- Django-based gift card application (target application used for testing)
- Web browser (to test the payload and view exposed credentials)
- Manual source code review
