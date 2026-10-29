# Investigation 01 — Initial Reconnaissance and Service Enumeration

## 1. Objective

Identify network services exposed by the Metasploitable 2 target within the authorised, isolated VirtualBox lab. The results were used to identify areas for further investigation.

## 2. Scope and Environment

- **Target:** Metasploitable 2 intentionally vulnerable training VM
- **Target IP:** `192.168.41.8`
- **Network:** Isolated VirtualBox host-only network
- **Tool:** Nmap 7.99
- **Scan:** TCP service/version detection with default NSE scripts

Testing was limited to the designated lab target. No external systems were in scope.

## 3. Methodology

The following command was used to identify accessible TCP services and gather preliminary service information:

`nmap -sV -sC -oN initial-scan.txt 192.168.41.8`

The scan results were reviewed to identify exposed services and potential areas requiring targeted validation.

## 4. Results

Nmap identified **23 open TCP ports**. The detected services included:

- File transfer: FTP
- Remote access: SSH, Telnet, and legacy remote login services
- Web services: HTTP and application-server services
- File sharing: SMB
- Name and network services: DNS and RPC
- Database services: MySQL and PostgreSQL
- Other services: Java RMI, VNC, NFS, and an IRC service

Several findings warranted additional investigation:

- Anonymous FTP access was permitted.
- SMB message signing was reported as disabled.
- The SMTP service was reported to support SSLv2.
- Multiple services advertised legacy software versions.
- Nmap identified a service on TCP port 1524 as a bindshell.

These results indicate a broad exposed service surface on the intentionally vulnerable target. They are initial observations, not proof that every service is exploitable.

## 5. Security Implications

An unnecessarily broad service surface can increase the number of potential entry points available to an attacker. Legacy services, weak access controls, and insecure protocol configurations may increase risk, depending on the service configuration and network exposure.

The initial scan alone does not establish the practical impact of each observation. Further investigation is needed to validate individual weaknesses and determine their severity.

## 6. Evidence

- [Sanitised Nmap initial scan summary](../evidence/01-nmap-initial-scan-summary.md)

The linked summary contains the relevant findings without publishing the full raw scan output or unnecessary host-identifying metadata.

## 7. Recommendations

- Disable services that are not required.
- Restrict access to necessary services using network controls.
- Replace unsupported software and insecure protocols with supported alternatives.
- Review authentication and anonymous-access settings.
- Investigate notable findings through separate, controlled tests.
- Document the evidence, limitations, remediation, and retest outcome for each validated issue.

## 8. Limitations

This was an initial reconnaissance activity. It was not a full vulnerability assessment, and no claim is made that all discovered services were tested for exploitation.

The results apply only to the isolated training environment and should not be interpreted as an assessment of a production system.

## 9. Conclusion

The reconnaissance scan identified 23 open TCP ports and several areas for follow-up testing. The results provided a baseline for the subsequent investigations in this portfolio.

