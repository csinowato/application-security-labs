# Application Security Labs
This repository contains a collection of vulnerability writeups based on assignments completed in 2020 as part of a graduate-level application security course, with documentation updated and expanded in 2025.

The labs were based on a custom gift card application written in C, with an additional Django web version. Each assignment involved identifying and exploiting vulnerabilities via crafted gift card files and implementing fixes for the underlying issues.

**Note:** The application's source code is not included, as it was developed by instructors at the university and provided for educational use only.

## Table of Contents
- [Signed Integer Overflow](./signed-integer-overflow/)
- [Out-of-Bounds Register](./out-of-bounds-register/)
- [Infinite Loop](./infinite-loop/)
- [XSS](./xss/)
- [CSRF](./csrf/)
- [SQL Injection](./sql-injection/)
- [Reused Salt](./reused-salt/)
- [Dictionary Attack](./dictionary-attack/)

## Summary of Exploits
Each vulnerability was discovered by analyzing how the application parsed and handled gift card files, then crafting malicious inputs to exploit weaknesses in memory handling, input validation, and application behavior.

Key vulnerabilities identified include:
- Signed integer overflow due to lack of validation checks before memory allocation
- Out-of-bounds register access leading to buffer overflow
- Unchecked loop termination leading to an infinite loop and denial of service
- Improper input handling leading to XSS and CSRF
- SQL injection via field manipulation
- Weak password storage with a reused salt

## Tech Stack
- C: core application for memory vulnerability labs
- Django (Python): web version of the application for XSS, CSRF, and SQL injection testing
- SQL:  used for injection payloads and database queries
- HTML: used for crafting CSRF exploit scripts
