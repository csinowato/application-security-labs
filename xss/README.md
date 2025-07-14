# Cross-Site Scripting (XSS)

## Summary
This section covers a reflected cross-site scripting (XSS) vulnerability in a web-based version of the gift card application, implemented using Django. Unlike some of the other vulnerabilities, which targeted the original C-based implementation (`giftcardreader.c`), this issue arises within the Python-based web interface and its HTML components.

To exploit this vulnerability, I injected a `<script>` tag into a URL parameter, which was reflected by the application without sanitization.

**Note:** The original source files were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerability occurs in the Django template `item-single.html`, which is responsible for displaying gift card details. The page reflects unsanitized input from the URL directly into the HTML, which allows for JavaScript injection.

## Vulnerability Details
The vulnerability was caused by the improper use of the `|safe` tag in `item-single.html`.
```
<p>Endorsed by {{director|safe}}!</p>
```
`|safe` means that the input is trusted, so malicious symbols and tags don't get automatically escaped. This makes it vulnerable to XSS attacks.

## Steps to Reproduce
To exploit the XSS vulnerability, I added a `<script>` tag directly to the URL and loaded the following page in the browser:
```
http://127.0.0.1:8000/buy/?director=<script>alert(1)</script>
```

[View screenshot of the reflected XSS attack](reflected-xss.png)

*The screenshot above shows the alert that was triggered in the browser by the injected script*

## Impact
Exploiting this vulnerability caused the application to execute arbitrary JavaScript in the browser. In a real-world context, this could be exploited further to steal session cookies or impersonate users.

## Mitigation
By default, Django escapes user input to prevent XSS, but the `|safe` filter disables this protection. So to fix this issue, I removed the `|safe` tag, therefore allowing Django to sanitize user inputs and prevent XSS.
```
<p>Endorsed by {{director}}!</p>
```

## Tools and Artifacts Used
- Web browser (used to craft and test the malicious URL payload)
- Django-based gift card application (target application used for testing)
- Manual source code review (used to identify the use of the `|safe` filter in the HTML)
