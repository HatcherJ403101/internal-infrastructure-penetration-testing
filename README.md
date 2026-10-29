# Internal Infrastructure Penetration Testing

A hands-on infrastructure and penetration-testing portfolio project conducted in an authorised, isolated VirtualBox lab. This project documents reconnaissance, service enumeration, access-control testing, web application vulnerabilities, evidence collection, and remediation recommendations.

The focus is on demonstrating a structured, evidence-led testing process: identifying potential weaknesses, validating what can be demonstrated, documenting limitations, and recommending practical security improvements.

## Project Overview

The assessment uses an intentionally vulnerable Metasploitable 2 virtual machine as the target and Kali Linux as the testing system.

The project covers:

- Network reconnaissance and service enumeration.
- Anonymous FTP and SMB access testing.
- WebDAV access-control and file-write testing.
- HTTP server version disclosure and security-header review.
- DVWA SQL injection input handling.
- DVWA operating-system command injection.
- Evidence sanitisation, investigation reporting, and remediation guidance.

All testing was conducted within the designated training environment. The findings are not an assessment of a production system.

## Lab Environment

| Component | Details |
|---|---|
| Testing system | Kali Linux |
| Target system | Metasploitable 2 |
| Virtualisation | Oracle VirtualBox |
| Network isolation | Host-only lab network |
| Target address | `192.168.41.8` — private lab address |
| Scope | Designated training VM only |

The lab was configured to keep the target isolated from external systems during testing. The private IP addresses documented in this repository refer to the training environment.

## Investigations

### 01 — Initial Reconnaissance and Service Enumeration

Performed initial Nmap service discovery and version detection to identify exposed TCP services and establish the scope for further investigation.

The results provide an initial view of the target's attack surface; service discovery alone does not prove that every identified service is exploitable.

[Read the investigation](investigations/01-initial-reconnaissance.md) · [View the sanitised Nmap summary](evidence/01-nmap-initial-scan-summary.md)

### 02 — Anonymous FTP Access

Tested anonymous FTP authentication and inspected the accessible directory listing.

Anonymous authentication succeeded, but the test did not demonstrate sensitive file disclosure.

[Read the investigation](investigations/02-anonymous-ftp-access.md)

### 03 — Samba Anonymous Write Access

Enumerated SMB shares and tested unauthenticated access to the `tmp` share.

The investigation confirmed that an anonymous user could create a test file. It did not demonstrate code execution or sensitive data disclosure.

[Read the investigation](investigations/03-samba-anonymous-write-access.md)

### 04 — Unauthenticated WebDAV Write Access

Examined WebDAV capabilities and tested unauthenticated file creation and deletion.

The test confirmed that a file could be uploaded without authentication and then removed successfully. The investigation did not demonstrate code execution.

[Read the investigation](investigations/04-webdav-unauthenticated-write.md)

### 05 — Legacy Web-Server Version Disclosure

Reviewed HTTP response headers and identified disclosure of legacy Apache and PHP version information.

The report distinguishes version disclosure from the wider security implications of running outdated software.

[Read the investigation](investigations/05-legacy-web-server-version-disclosure.md)

### 06 — Missing HTTP Security Headers

Reviewed the target's HTTP response for common browser security headers.

The investigation documented the absence of selected headers in the observed response. HSTS was not assessed because the test used HTTP rather than HTTPS.

[Read the investigation](investigations/06-missing-http-security-headers.md)

### 07 — DVWA SQL Injection Input Handling

Compared input handling at different DVWA security levels.

At Low security, a single-quote input produced a visible database syntax error. The investigation also reviewed the relevant source code and compared the observed behaviour with High security.

The testing did not demonstrate successful unauthorised data retrieval.

[Read the investigation](investigations/07-dvwa-sql-injection.md)

### 08 — DVWA Operating-System Command Injection

Compared command-injection behaviour at Low, Medium, and High security.

At Low security, the input `192.168.41.8; id` caused an additional operating-system command to execute under the `www-data` account. The tested Medium-security inputs produced no visible additional command output, while the tested crafted input at High security was rejected as an invalid IP address.

The report explains the limitations of the evidence and does not claim that the Medium and High security levels are comprehensively secure.

[Read the investigation](investigations/08-dvwa-command-injection.md)

## Methodology

The project follows a structured, evidence-led workflow:

1. **Reconnaissance:** Identify reachable services and collect initial service information.
2. **Enumeration:** Examine relevant services, shares, methods, and application functionality.
3. **Validation:** Test specific suspected weaknesses within the authorised lab.
4. **Evidence collection:** Record commands, responses, and screenshots that support the observations.
5. **Analysis:** Distinguish confirmed behaviour from potential impact and unverified assumptions.
6. **Reporting:** Document findings, limitations, and practical remediation measures.
7. **Retesting:** Define criteria that can be used to assess whether a weakness has been addressed.

## Evidence and Documentation

Supporting evidence and methodology are documented alongside the investigations.

- [Evidence directory and handling guidance](evidence/README.md)
- [Sanitised Nmap scan summary](evidence/01-nmap-initial-scan-summary.md)
- [Curated Kali command log](evidence/kali-command-log.md)

The repository uses curated summaries and reviewed screenshots rather than publishing unreviewed raw scan output or a complete terminal history.

Evidence is presented in context. A screenshot supports the specific behaviour visible in the image, but it may not independently establish every test condition. The investigation reports document these limitations where relevant.

## Key Findings

The investigations have documented the following observations in the training environment:

| Area | Observed result |
|---|---|
| FTP | Anonymous authentication was permitted; sensitive file disclosure was not demonstrated. |
| SMB | Anonymous file creation in the `tmp` share was confirmed. |
| WebDAV | Unauthenticated file creation and subsequent deletion were confirmed. |
| HTTP version disclosure | Legacy Apache and PHP version information was exposed in response headers. |
| HTTP security headers | Selected browser security headers were absent from the observed response. |
| SQL injection | A malformed input triggered a visible database syntax error at DVWA Low security; unauthorised data retrieval was not demonstrated. |
| Command injection | Additional command execution under `www-data` was confirmed at DVWA Low security. |

These results apply to the intentionally vulnerable lab. Their real-world severity would depend on factors such as network exposure, application configuration, process privileges, accessible resources, and other security controls.

## Remediation Themes

The findings highlight several important defensive practices:

- Restrict anonymous access and unnecessary write permissions.
- Disable services and network methods that are not required.
- Maintain supported software versions and apply security updates.
- Use parameterised database queries and safe input-handling practices.
- Avoid constructing shell commands by concatenating user-controlled input.
- Run application processes with the minimum privileges necessary.
- Configure appropriate HTTP security headers for the application and deployment.
- Review related code paths and retest after remediation.

## Limitations

This is a learning and portfolio project, not a comprehensive security assessment or a penetration test of a production environment.

Testing was limited to selected services, application functions, and inputs. Not every exposed service was fully investigated. The SQL injection investigation demonstrated a database error, not successful unauthorised data retrieval. The command-injection investigation confirmed execution at Low security, but the limited Medium and High tests do not establish comprehensive protection.

No claim is made that every possible vulnerability or attack path was identified, or that all potential impacts were demonstrated.

## Project Status

Eight investigations have been documented, covering reconnaissance, FTP, SMB, WebDAV, HTTP configuration, SQL injection input handling, and operating-system command injection.

The repository includes individual investigation reports, a sanitised Nmap summary, a curated command log, and supporting evidence.

This project demonstrates practical lab testing, technical documentation, evidence handling, and security-focused remediation analysis.
