# Testing Guide

## Summary

Sensitive data must be protected when it is transmitted through the network. If data is transmitted over HTTPS or encrypted in another way the protection mechanism must not have limitations or vulnerabilities, as explained in the broader article [Testing for Weak Transport Layer Security](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/01-Testing_for_Weak_Transport_Layer_Security) and in other OWASP documentation:

-   [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure).
-   [OWASP ASVS - Verification V9](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x17-V9-Communications.md).
-   [Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

As a rule of thumb if data must be protected when it is stored, this data must also be protected during transmission. Some examples for sensitive data are:

-   Information used in authentication (e.g. Credentials, PINs, Session identifiers, Tokens, Cookies…)
-   Information protected by laws, regulations or specific organizational policy (e.g. Credit Cards, Customers data)

If the application transmits sensitive information via unencrypted channels - e.g. HTTP - it is considered a security risk. Some examples are Basic authentication which sends authentication credentials in plain-text over HTTP, form based authentication credentials sent via HTTP, or plain-text transmission of any other information considered sensitive due to regulations, laws, organizational policy or application business logic.

Examples for Personal Identifying Information (PII) are:

-   Social security numbers
-   Bank account numbers
-   Passport information
-   Healthcare related information
-   Medical insurance information
-   Student information
-   Credit and debit card numbers
-   Drivers license and State ID information

## Test Objectives

-   Identify sensitive information transmitted through the various channels.
-   Assess the privacy and security of the channels used.

## How to Test

Various types of information that must be protected, could be transmitted by the application in clear text. It is possible to check if this information is transmitted over HTTP instead of HTTPS, or whether weak ciphers are used. See more information about insecure transmission of credentials [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) or [Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

### Example 1: Basic Authentication over HTTP

A typical example is the usage of Basic Authentication over HTTP. When using Basic Authentication, user credentials are encoded rather than encrypted, and are sent as HTTP headers. In the example below the tester uses [curl](https://curl.haxx.se/) to test for this issue. Note how the application uses Basic authentication, and HTTP rather than HTTPS

```
$ curl -kis http://example.com/restricted/
HTTP/1.1 401 Authorization Required
Date: Fri, 01 Aug 2013 00:00:00 GMT
WWW-Authenticate: Basic realm="Restricted Area"
Accept-Ranges: bytes Vary:
Accept-Encoding Content-Length: 162
Content-Type: text/html

<html><head><title>401 Authorization Required</title></head>
<body bgcolor=white> <h1>401 Authorization Required</h1>  Invalid login credentials!  </body></html>
```

### Example 2: Form-Based Authentication Performed over HTTP

Another typical example is authentication forms which transmit user authentication credentials over HTTP. In the example below one can see HTTP being used in the `action` attribute of the form. It is also possible to see this issue by examining the HTTP traffic with an interception proxy.

```
<form action="http://example.com/login">
    <label for="username">User:</label> <input type="text" id="username" name="username" value=""/><br />
    <label for="password">Password:</label> <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
</form>
```

### Example 3: Cookie Containing Session ID Sent over HTTP

The Session ID Cookie must be transmitted over protected channels. If the cookie does not have the [secure flag](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes) set, it is permitted for the application to transmit it unencrypted. Note below the setting of the cookie is done without the Secure flag, and the entire log in process is performed in HTTP and not HTTPS.

```
https://secure.example.com/login

POST /login HTTP/1.1
Host: secure.example.com
[...]
Referer: https://secure.example.com/
Content-Type: application/x-www-form-urlencoded
Content-Length: 188

HTTP/1.1 302 Found
Date: Tue, 03 Dec 2013 21:18:55 GMT
Server: Apache
Set-Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA; expires=Fri, 01-Jan-2014 00:00:00 GMT; path=/; domain=example.com; httponly
Location: private/
Content-Length: 0
Content-Type: text/html
```

```
http://example.com/private

GET /private HTTP/1.1
Host: example.com
[...]
Referer: https://secure.example.com/login
Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;

HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 730
Date: Tue, 25 Dec 2013 00:00:00 GMT
```

### Example 4: Testing Password Sensitive Information in Source Code or Logs

Use one of the following techniques to search for senstive information.

Checking if password or encyrption key is hardcoded in the source code or configuration files.

`grep -r –E "Pass | password | pwd |user | guest| admin | encry | key | decrypt | sharekey " ./PathToSearch/`

Checking if logs or source code may contain phone number, email address, ID or any other PII. Change the regular expression based on the format of the PII.

`grep -r " {2\}[0-9]\{6\} " ./PathToSearch/`

## Tools

-   [curl](https://curl.haxx.se/)
-   [grep](http://man7.org/linux/man-pages/man1/egrep.1.html)
-   [Identity Finder](https://download.cnet.com/Identity-Finder-Free-Edition/3000-2144_4-10906766.html)
-   [Wireshark](https://www.wireshark.org/)
-   [TCPDUMP](https://www.tcpdump.org/)

---

# Test Documentation

Document here every action performed for this test.