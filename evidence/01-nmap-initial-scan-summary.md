# Initial Nmap Service Enumeration

## Assessment Context

- **Target:** Metasploitable 2 training VM
- **Target address:** `192.168.41.8` (private lab network)
- **Tool:** Nmap 7.99
- **Scan options:** Service/version detection and default NSE scripts (`-sV -sC`)
- **Scope:** Authorised, isolated VirtualBox host-only lab

## Summary

The target was reachable, and Nmap identified 23 open TCP ports. Service detection revealed a range of legacy services, including FTP, SSH, Telnet, SMTP, DNS, HTTP, SMB, remote administration services, databases and application servers.

Notable initial observations included:

- Anonymous FTP login was permitted.
- SMB message signing was reported as disabled.
- SSLv2 support was reported on the SMTP service.
- Several services advertised legacy software versions.
- A service was identified on TCP port 1524 as a bindshell.

These observations informed subsequent, targeted investigations. They do not, by themselves, establish that every identified service is exploitable.

## Limitations

This was an initial reconnaissance scan, not a complete vulnerability assessment. Individual services require separate validation, and severity should be based on demonstrated behaviour and potential impact.

## Safety and Sanitisation

The assessment was conducted against an intentionally vulnerable training VM in an isolated lab. The address shown is a private lab address, not a public target.

This summary intentionally excludes the raw scan output, host-key fingerprints, MAC address and unnecessary service metadata.
