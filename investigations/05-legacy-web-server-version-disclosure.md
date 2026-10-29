# Investigation 05 — Legacy Web-Server Version Disclosure

## 1. Objective

Review the target's HTTP response headers to determine whether the web server discloses software version information that could assist service identification and subsequent security assessment.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM
- **Target IP:** `192.168.41.8`
- **Endpoint:** `/`
- **Port:** TCP 80 (HTTP)
- **Environment:** Authorised, isolated VirtualBox host-only lab

## 3. Methodology

The HTTP response headers were inspected using:

`curl -sS -D - -o /dev/null http://192.168.41.8/`

The request retrieved the response headers without downloading the page body.

## 4. Results

The server returned an HTTP `200 OK` response and disclosed software information in the response headers, including:

- Apache HTTP Server `2.2.8`
- PHP `5.2.4-2ubuntu5.10`

These versions are obsolete and no longer appropriate for a maintained production deployment.

## 5. Security Impact

Software version disclosure can help an attacker identify technologies and research vulnerabilities associated with particular releases. However, the presence of a version string alone does not prove that a specific vulnerability is exploitable.

The more significant concern is the use of unsupported software that may lack current security fixes. The actual exposure depends on the deployed configuration, available patches, and network accessibility.

**Assessment:** Informational for version disclosure alone. The use of unsupported components warrants separate risk assessment and remediation.

## 6. Evidence

Evidence should be uploaded only after review and sanitisation.

Planned evidence: a sanitised HTTP response-header capture showing the relevant `Server` and `X-Powered-By` headers. Remove unrelated response data and any sensitive information before publication.

## 7. Recommendations

- Upgrade Apache, PHP, and the underlying operating system to supported versions.
- Apply security updates through a maintained software distribution.
- Remove unnecessary version details from response headers where practical.
- Maintain an inventory of web-server components and their support status.
- Validate specific vulnerabilities separately rather than relying on version strings alone.

## 8. Retest Criteria

Repeat the HTTP header inspection after remediation.

- **Pass:** The server runs supported software, and unnecessary version details are minimised.
- **Fail:** Unsupported components remain deployed without an approved, documented exception.

Reducing version disclosure alone is not sufficient to remediate vulnerabilities in outdated software.

## 9. Limitations

This investigation reviewed HTTP response headers only. It did not establish exploitation of Apache or PHP vulnerabilities.

