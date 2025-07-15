# Cross-Site Request Forgery (CSRF)

## Summary
This section covers a cross-site request forgery (CSRF) vulnerability in a web-based version of the gift card application implemented using Django. Unlike some of the other vulnerabilities, which targeted the original C-based implementation, this issue arises within the Python-based web interface and its HTML components.

To exploit this vulnerability, I created a script called `attack.html` that sends a `POST` request to `/gift/0` with the victim's session cookie and automatically submits the form. This allows the attacker to receive a gift card from the victim without the victim's consent.

**Note:** The original source files were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerable component was the `gift.html` template, which was missing a CSRF token in its form. This made the application vulnerable to CSRF attacks.

## Vulnerability Details
The form in `gift.html` didn't include a CSRF token, allowing an attacker to craft a malicious HTML page that sends an unauthorized POST request on behalf of a logged-in user.
```
<form action="/gift/{{ prod_num }}" method="post">
    <!-- form inputs -->
</form>
```

## Steps to Reproduce
The code snippet below is a simplified version of the form in the `attack.html` script that was used to perform the CSRF attack.
```
<form action=".../gift/0" method="POST">
    <input type="hidden" id="amount" name="amount" value="750"/>
    <input type="hidden" name="username" value="attacker"/>
</form>
<script>document.forms[0].submit()</script>
```

The CSRF attack was confirmed by observing the browser automatically sending the victim's session cookie in the forged `POST` request. Using Chrome DevTools, I verified that the request was made to the target endpoint and was accepted by the server.

Below is a simplified version of the forged request:
```
POST /gift/0 HTTP/1.1
Cookie: sessionid=victim_session_cookie
...
```

**Note:** Since the attack had no visible frontend response and was validated through network and cookie inspection, no screenshot was taken at the time.

## Impact
Exploiting this vulnerability allowed an attacker to perform an unauthorized action (i.e. a gift card transfer) on behalf of a logged-in user. In a real-world scenario, this vulnerability could allow attackers to perform other unintended actions on behalf of authenticated users without their knowledge. This includes things such as unauthorized monetary transfers, data changes, or account takeover.

## Mitigation
To fix this vulnerability, I added a CSRF token (`{% csrf_token %}`) in the Django template `gift.html`. `{% csrf_token %}` is Django's built in CSRF protection, so including it ensures that a CSRF token is sent with every POST request for this form.
```
<form action="/gift/{{ prod_num }}" method="post">
    <!-- form inputs -->
    {% csrf_token %}
</form>
```
In `views.py`, I also added `@csrf_protect`. This further enforces CSRF protection in the `gift_card_view` function, so if the attacker doesn't include the correct token, it results in a 403 forbidden response stating that the CSRF verification failed.

The code below has been simplified for clarity.
```
@csrf_protect
def gift_card_view(request):
    # Handles form data and page rendering
    ...
```

## Tools and Artifacts Used
- `attack.html` (custom HTML page used to deliver the CSRF payload)
- Chrome DevTools (used to inspect the forged request and verify that the victim's session cookie was sent)
- Django-based gift card application (target application used for testing)
- Browser session management (used to simulate victim and attacker sessions)
- Manual source code review
