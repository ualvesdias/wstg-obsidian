# Testing Guide

## Summary

Client-side SQL injection occurs when an application implements the [Web SQL Database](https://www.w3.org/TR/webdatabase/) technology and doesn’t properly validate the input nor parametrize its query variables. This database is manipulated by using JavaScript (JS) API calls, such as `openDatabase()`, which creates or opens an existing database.

## Test Objectives

The following test scenario will validate that proper input validation is conducted. If the implementation is vulnerable, the attacker can read, modify, or delete information stored within the database.

## How to Test

### Identify the Usage of Web SQL DB

If the tested application implements the Web SQL DB, the following three calls will be used in the client-side core:

-   `openDatabase()`
-   `transaction()`
-   `executeSQL()`

The code below shows an example of the APIs’ implementation:

```
var db = openDatabase(shortName, version, displayName, maxSize);

db.transaction(function(transaction) {
    transaction.executeSql('INSERT INTO LOGS (time, id, log) VALUES (?, ?, ?)', [dateTime, id, log]);
});
```

### Web SQL DB Injection

After confirming the usage of `executeSQL()`, the attacker is ready to test and validate the security of its implementation.

The Web SQL DB’s implementation is based on [SQLite’s syntax](https://www.sqlite.org/lang.html).

#### Bypassing Conditions

The following example shows how this could be exploited on the client-side:

```
// URL example: https://example.com/user#15
var userId = document.location.hash.substring(1,); // Grabs the ID without the hash -> 15

db.transaction(function(transaction){
    transaction.executeSQL('SELECT * FROM users WHERE user = ' + userId);
});
```

To return information for all the users, instead of only the user corresponding to the attacker, the following could be used: `15 OR 1=1` in the URL fragment.

For additional SQL Injection payloads, go to the [Testing for SQL Injection](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection) scenario.

## Remediation

Follow the same remediation from the [Testing for SQL Injection’s Remediation Section](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection#remediation).

## References

-   [W3C Web SQL Database](https://www.w3.org/TR/webdatabase/)
-   [Apple’s JavaScript Database Tutorial](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/SafariJSDatabaseGuide/UsingtheJavascriptDatabase/UsingtheJavascriptDatabase.html)
-   [Tutorialspoint HTML5 Web SQL Database](https://www.tutorialspoint.com/html5/html5_web_sql.htm)
-   [Portswigger’s Client-Side SQL Injection](https://portswigger.net/web-security/dom-based/client-side-sql-injection)

---

# Test Documentation

Document here every action performed for this test.