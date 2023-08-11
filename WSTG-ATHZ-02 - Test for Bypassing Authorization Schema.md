# Testing Guide

## Summary

This kind of test focuses on verifying how the authorization schema has been implemented for each role or privilege to get access to reserved functions and resources.

For every specific role the tester holds during the assessment and for every function and request that the application executes during the post-authentication phase, it is necessary to verify:

-   Is it possible to access that resource even if the user is not authenticated?
-   Is it possible to access that resource after the log-out?
-   Is it possible to access functions and resources that should be accessible to a user that holds a different role or privilege?

Try to access the application as an administrative user and track all the administrative functions.

-   Is it possible to access administrative functions if the tester is logged in as a non-admin user?
-   Is it possible to use these administrative functions as a user with a different role and for whom that action should be denied?

## Test Objectives

-   Assess if horizontal or vertical access is possible.

## How to Test

-   Access resources and conduct operations horizontally.
-   Access resources and conduct operations vertically.

### Testing for Horizontal Bypassing Authorization Schema

For every function, specific role, or request that the application executes, it is necessary to verify:

-   Is it possible to access resources that should be accessible to a user that holds a different identity with the same role or privilege?
-   Is it possible to operate functions on resources that should be accessible to a user that holds a different identity?

For each role:

1.  Register or generate two users with identical privileges.
2.  Establish and keep two different sessions active (one for each user).
3.  For every request, change the relevant parameters and the session identifier from token one to token two and diagnose the responses for each token.
4.  An application will be considered vulnerable if the responses are the same, contain same private data or indicate successful operation on other users’ resource or data.

For example, suppose that the `viewSettings` function is part of every account menu of the application with the same role, and it is possible to access it by requesting the following URL: `https://www.example.com/account/viewSettings`. Then, the following HTTP request is generated when calling the `viewSettings` function:

```
POST /account/viewSettings HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=USER_SESSION

username=example_user
```

Valid and legitimate response:

```
HTTP1.1 200 OK
[other HTTP headers]

{
  "username": "example_user",
  "email": "example@email.com",
  "address": "Example Address"
}
```

The attacker may try and execute that request with the same `username` parameter:

```
POST /account/viewCCpincode HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ATTACKER_SESSION

username=example_user
```

If the attacker’s response contain the data of the `example_user`, then the application is vulnerable for lateral movement attacks, where a user can read or write other user’s data.

### Testing for Vertical Bypassing Authorization Schema

A vertical authorization bypass is specific to the case that an attacker obtains a role higher than their own. Testing for this bypass focuses on verifying how the vertical authorization schema has been implemented for each role. For every function, page, specific role, or request that the application executes, it is necessary to verify if it is possible to:

-   Access resources that should be accessible only to a higher role user.
-   Operate functions on resources that should be operative only by a user that holds a higher or specific role identity.

For each role:

1.  Register a user.
2.  Establish and maintain two different sessions based on the two different roles.
3.  For every request, change the session identifier from the original to another role’s session identifier and evaluate the responses for each.
4.  An application will be considered vulnerable if the weaker privileged session contains the same data, or indicate successful operations on higher privileged functions.

#### Banking Site Roles Scenario

The following table illustrates the system roles on a banking site. Each role binds with specific permissions for the event menu functionality:

ROLE

PERMISSION

ADDITIONAL PERMISSION

Administrator

Full Control

Delete

Manager

Modify, Add, Read

Add

Staff

Read, Modify

Modify

Customer

Read Only

 

The application will be considered vulnerable if the:

1.  Customer could operate administrator, manager or staff functions;
2.  Staff user could operate manager or administrator functions;
3.  Manager could operate administrator functions.

Suppose that the `deleteEvent` function is part of the administrator account menu of the application, and it is possible to access it by requesting the following URL: `https://www.example.com/account/deleteEvent`. Then, the following HTTP request is generated when calling the `deleteEvent` function:

```
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ADMINISTRATOR_USER_SESSION

EventID=1000001
```

The valid response:

```
HTTP/1.1 200 OK
[other HTTP headers]

{"message": "Event was deleted"}
```

The attacker may try and execute the same request:

```
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=CUSTOMER_USER_SESSION

EventID=1000002
```

If the response of the attacker’s request contains the same data `{"message": "Event was deleted"}` the application is vulnerable.

#### Administrator Page Access

Suppose that the administrator menu is part of the administrator account.

The application will be considered vulnerable if any role other than administrator could access the administrator menu. Sometimes, developers perform authorization validation at the GUI level only, and leave the functions without authorization validation, thus potentially resulting in a vulnerability.

### Testing for Access to Administrative Functions

For example, suppose that the `addUser` function is part of the administrative menu of the application, and it is possible to access it by requesting the following URL `https://www.example.com/admin/addUser`.

Then, the following HTTP request is generated when calling the `addUser` function:

```
POST /admin/addUser HTTP/1.1
Host: www.example.com
[...]

userID=fakeuser&role=3&group=grp001
```

Further questions or considerations would go in the following direction:

-   What happens if a non-administrative user tries to execute that request?
-   Will the user be created?
-   If so, can the new user use their privileges?

### Testing for Access to Resources Assigned to a Different Role

Various applications setup resource controls based on user roles. Let’s take an example resumes or CVs (curriculum vitae) uploaded on a careers form to an S3 bucket.

As a normal user, try accessing the location of those files. If you are able to retrieve them, modify them, or delete them, then the application is vulnerable.

### Testing for Special Request Header Handling

Some applications support non-standard headers such as `X-Original-URL` or `X-Rewrite-URL` in order to allow overriding the target URL in requests with the one specified in the header value.

This behavior can be leveraged in a situation in which the application is behind a component that applies access control restriction based on the request URL.

The kind of access control restriction based on the request URL can be, for example, blocking access from Internet to an administration console exposed on `/console` or `/admin`.

To detect the support for the header `X-Original-URL` or `X-Rewrite-URL`, the following steps can be applied.

#### 1. Send a Normal Request without Any X-Original-Url or X-Rewrite-Url Header

```
GET / HTTP/1.1
Host: www.example.com
[...]
```

#### 2. Send a Request with an X-Original-Url Header Pointing to a Non-Existing Resource

```
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
[...]
```

#### 3. Send a Request with an X-Rewrite-Url Header Pointing to a Non-Existing Resource

```
GET / HTTP/1.1
Host: www.example.com
X-Rewrite-URL: /donotexist2
[...]
```

If the response for either request contains markers that the resource was not found, this indicates that the application supports the special request headers. These markers may include the HTTP response status code 404, or a “resource not found” message in the response body.

Once the support for the header `X-Original-URL` or `X-Rewrite-URL` was validated then the tentative of bypass against the access control restriction can be leveraged by sending the expected request to the application but specifying a URL “allowed” by the front-end component as the main request URL and specifying the real target URL in the `X-Original-URL` or `X-Rewrite-URL` header depending on the one supported. If both are supported then try one after the other to verify for which header the bypass is effective.

#### 4. Other Headers to Consider

Often admin panels or administrative related bits of functionality are only accessible to clients on local networks, therefore it may be possible to abuse various proxy or forwarding related HTTP headers to gain access. Some headers and values to test with are:

-   Headers:
    -   `X-Forwarded-For`
    -   `X-Forward-For`
    -   `X-Remote-IP`
    -   `X-Originating-IP`
    -   `X-Remote-Addr`
    -   `X-Client-IP`
-   Values
    -   `127.0.0.1` (or anything in the `127.0.0.0/8` or `::1/128` address spaces)
    -   `localhost`
    -   Any [RFC1918](https://tools.ietf.org/html/rfc1918) address:
        -   `10.0.0.0/8`
        -   `172.16.0.0/12`
        -   `192.168.0.0/16`
    -   Link local addresses: `169.254.0.0/16`

Note: Including a port element along with the address or hostname may also help bypass edge protections such as web application firewalls, etc. For example: `127.0.0.4:80`, `127.0.0.4:443`, `127.0.0.4:43982`

## Remediation

Employ the least privilege principles on the users, roles, and resources to ensure that no unauthorized access occurs.

## Tools

-   [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
    -   [ZAP add-on: Access Control Testing](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)
-   [Port Swigger Burp Suite](https://portswigger.net/burp)
    -   [Burp extension: AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix/)
    -   [Burp extension: Autorize](https://github.com/Quitten/Autorize)

## References

[OWASP Application Security Verification Standard 4.0.1](https://github.com/OWASP/ASVS/tree/master/4.0), v4.0.1-1, v4.0.1-4, v4.0.1-9, v4.0.1-16

---

# Test Documentation

Document here every action performed for this test.