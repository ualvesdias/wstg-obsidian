# Testing Guide

## Summary

A web server commonly hosts several web applications on the same IP address, referring to each application via the virtual host. In an incoming HTTP request, web servers often dispatch the request to the target virtual host based on the value supplied in the Host header. Without proper validation of the header value, the attacker can supply invalid input to cause the web server to:

-   dispatch requests to the first virtual host on the list
-   cause a redirect to an attacker-controlled domain
-   perform web cache poisoning
-   manipulate password reset functionality

## Test Objectives

-   Assess if the Host header is being parsed dynamically in the application.
-   Bypass security controls that rely on the header.

## How to Test

Initial testing is as simple as supplying another domain (i.e. `attacker.com`) into the Host header field. It is how the web server processes the header value that dictates the impact. The attack is valid when the web server processes the input to send the request to an attacker-controlled host that resides at the supplied domain, and not to an internal virtual host that resides on the web server.

```
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

In the simplest case, this may cause a 302 redirect to the supplied domain.

```
HTTP/1.1 302 Found
[...]
Location: http://www.attacker.com/login.php

```

Alternatively, the web server may send the request to the first virtual host on the list.

### X-Forwarded Host Header Bypass

In the event that Host header injection is mitigated by checking for invalid input injected via the Host header, you can supply the value to the `X-Forwarded-Host` header.

```
GET / HTTP/1.1
Host: www.example.com
X-Forwarded-Host: www.attacker.com
...
```

Potentially producing client-side output such as:

```
...
<link src="http://www.attacker.com/link" />
...
```

Once again, this depends on how the web server processes the header value.

### Web Cache Poisoning

Using this technique, an attacker can manipulate a web-cache to serve poisoned content to anyone who requests it. This relies on the ability to poison the caching proxy run by the application itself, CDNs, or other downstream providers. As a result, the victim will have no control over receiving the malicious content when requesting the vulnerable application.

```
GET / HTTP/1.1
Host: www.attacker.com
...
```

The following will be served from the web cache, when a victim visits the vulnerable application.

```
...
<link src="http://www.attacker.com/link" />
...
```

### Password Reset Poisoning

It is common for password reset functionality to include the Host header value when creating password reset links that use a generated secret token. If the application processes an attacker-controlled domain to create a password reset link, the victim may click on the link in the email and allow the attacker to obtain the reset token, thus resetting the victim’s password.

```
... Email snippet ...

Click on the following link to reset your password:

http://www.attacker.com/index.php?module=Login&action=resetPassword&token=<SECRET_TOKEN>

... Email snippet ...
```

## References

-   [What is a Host Header Attack?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
-   [Host Header Attack](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
-   [Practical HTTP Host Header Attacks](https://www.skeletonscribe.net/2013/05/practical-http-host-header-attacks.html)

---

# Test Documentation

Document here every action performed for this test.