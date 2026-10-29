# Investigation 07 — DVWA SQL Injection Input Handling

## 1. Objective

Assess how the Damn Vulnerable Web Application (DVWA) SQL Injection module handles user-supplied input at different security levels and whether database errors are exposed to the user.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM running DVWA
- **Target address:** `192.168.41.8` (private lab network)
- **Application:** DVWA SQL Injection module
- **Environment:** Authorised, isolated VirtualBox host-only lab

Testing was limited to the intentionally vulnerable training application.

## 3. Methodology

The SQL Injection module was tested using a normal numeric input and a single quote character at the Low and High security levels.

The application responses were compared, and the relevant source code was reviewed to understand how input was handled.

No attempt was made to access data beyond the intended test records.

## 4. Results

### Low security

A normal numeric input returned the expected user record.

Submitting a single quote caused a visible MySQL syntax error. This demonstrates that malformed user input can trigger a database error that is returned to the application user.

The reviewed source code constructed the SQL query by concatenating the supplied value into the query string without the input protections used at the High level.

### High security

The same normal numeric input returned the expected record.

The single quote input did not produce the same visible database error. The reviewed source code included input escaping and a numeric-input check before constructing the query.

These observations show differences in input handling between the two application security levels.

**Confirmed:** The Low-level input triggered a visible SQL syntax error, and the reviewed code showed direct query construction from user input.

**Not demonstrated:** Successful SQL injection resulting in unauthorised data retrieval, authentication bypass, or modification of database records.

## 5. Security Impact

Returning raw database errors can disclose implementation details and assist further testing. Constructing SQL statements from user-controlled input can also create SQL injection risk when input is not safely handled.

The observed error alone does not prove that an attacker can retrieve or modify unauthorised data. The risk assessment should reflect the confirmed behaviour and the application's intentionally vulnerable training context.

## 6. Evidence

Evidence should be uploaded only after review and sanitisation.

Planned evidence:

- A sanitised screenshot showing the Low-level input and resulting SQL error.
- A sanitised screenshot showing the corresponding High-level behaviour.
- Optional, carefully selected source-code excerpts illustrating the different input-handling approaches.

Before publishing screenshots, remove session cookies, session identifiers, credentials, and unrelated browser information.

## 7. Recommendations

- Use parameterised queries or prepared statements rather than concatenating user input into SQL.
- Validate input according to the expected data type and business rules.
- Apply database permissions appropriate to the application's requirements.
- Return generic error messages to users and keep diagnostic details in protected server-side logs.
- Retest the application using representative valid and invalid inputs.

## 8. Retest Criteria

Repeat the input-handling tests after remediation.

- **Pass:** User input cannot alter the intended SQL query structure, and raw database errors are not exposed to users.
- **Fail:** Crafted input can change query behaviour or detailed database errors are returned to unauthorised users.

## 9. Limitations

This investigation covered a small number of inputs and two DVWA security levels. It did not establish successful SQL injection or comprehensively assess the application's database security.

