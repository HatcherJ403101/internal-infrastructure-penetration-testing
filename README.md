# Internal Infrastructure Penetration Testing

A practical internal network and infrastructure penetration-testing project conducted in an authorised, isolated VirtualBox lab. This portfolio documents service enumeration, access-control weaknesses, web application testing, evidence handling, and remediation recommendations.

## Project Overview

The assessment uses an intentionally vulnerable Metasploitable 2 virtual machine as the target and Kali Linux as the testing system. Both systems communicate over an isolated host-only network.

The objective is to practise a structured penetration-testing workflow: identify exposed services, validate security weaknesses, record observed behaviour, assess potential impact, and recommend appropriate remediation.

## Lab Environment

| Component | Details |
|---|---|
| Testing system | Kali Linux |
| Target system | Metasploitable 2 |
| Virtualisation | Oracle VirtualBox |
| Network isolation | Host-only network |
| Target address | `192.168.41.8` (private lab address) |
| Scope | Designated training VM only |

All testing described in this repository was performed within the authorised lab environment. The findings should not be interpreted as an assessment of a production system.

## Investigations

### 01 — Initial Reconnaissance and Service Enumeration

Identified exposed TCP services and recorded initial observations to guide subsequent testing.

[Read the investigation](investigations/01-initial-reconnaissance.md) · [Sanitised Nmap summary](evidence/01-nmap-initial-scan-summary.md)

### 02 — Anonymous FTP Access

Tested anonymous FTP authentication and inspected the accessible directory listing.

[Read the investigation](investigations/02-anonymous-ftp-access.md)

### 03 — Samba Anonymous Write Access

Enumerated SMB shares and confirmed that an unauthenticated user could create a file in the `tmp` share.

[Read the investigation](investigations/03-samba-anonymous-write-access.md)

### 04 — Unauthenticated WebDAV Write Access

Tested WebDAV methods and confirmed unauthenticated file creation, followed by successful cleanup of the test file.

[Read the investigation](investigations/04-webdav-unauthenticated-write.md)

### 05 — Legacy Web-Server Version Disclosure

Reviewed HTTP response headers and identified disclosure of legacy Apache and PHP version information.

[Read the investigation](investigations/05-legacy-web-server-version-disclosure.md)

### 06 — Missing HTTP Security Headers

Reviewed the HTTP response for common browser security headers and documented the limitations of the observation.

[Read the investigation](investigations/06-missing-http-security-headers.md)

### 07 — DVWA SQL Injection Input Handling

Compared input handling at different DVWA security levels and observed a database syntax error at Low security. Successful unauthorised data retrieval was not demonstrated.

[Read the investigation](investigations/07-dvwa-sql-injection.md)

### 08 — DVWA Operating-System Command Injection

Confirmed that crafted input executed an additional operating-system command at Low security, running as `www-data`. The tested inputs at Medium and High did not demonstrate successful additional command execution.

[Read the investigation](investigations/08-dvwa-command-injection.md)

## Methodology

The project follows a structured, evidence-led approach:

1. Identify reachable services and technologies.
2. Select relevant services or application functions for further testing.
3. Validate specific behaviours within the authorised lab.
4. Record observed results and distinguish confirmed impact from potential risk.
5. Recommend remediation and define criteria for retesting.

## Evidence and Reporting

Each investigation is documented separately and linked from this README. Supporting evidence is being curated for public release rather than publishing unreviewed raw output.

Before publication, evidence should be checked for credentials, session data, private host information, fingerprints, and other unnecessary identifying details.

See the [evidence directory](evidence/README.md) for the evidence-handling approach.

## Key Findings So Far

Testing has confirmed:

- Anonymous FTP authentication was permitted.
- Anonymous users could create files in an SMB share.
- Unauthenticated WebDAV file creation was possible.
- Legacy web-server and PHP versions were disclosed in HTTP headers.
- A malformed SQL input triggered a visible database error at DVWA Low security.
- Command injection was confirmed in DVWA at Low security, with command execution under the `www-data` account.

These results apply to the intentionally vulnerable training environment. Potential severity depends on deployment context, network exposure, permissions, and other controls.

## Remediation Themes

The observations highlight the importance of:

- Restricting anonymous access and unnecessary write permissions.
- Disabling unneeded services and network methods.
- Maintaining supported software versions.
- Using parameterised database queries and safe operating-system process execution.
- Applying appropriate HTTP security headers.
- Revalidating findings after remediation.

## Limitations

This is a learning and portfolio project, not a comprehensive security assessment. Some findings have only been validated in limited test cases. No claim is made that every exposed service was fully assessed or that successful exploitation was demonstrated for every finding.

