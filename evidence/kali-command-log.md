# Kali Linux Command Log

This document records the main commands used during the internal infrastructure penetration-testing assessment. It provides a curated, sanitised reference for the methodology documented in this repository.

All testing was conducted within an authorised, isolated VirtualBox lab using Kali Linux and an intentionally vulnerable Metasploitable 2 target.

## 1. Network Configuration and Verification

Commands used to inspect and configure Kali Linux network interfaces:

```bash
ip -br addr
ip route
nmcli device status
systemctl is-active NetworkManager
sudo nmcli device connect eth0
nmcli connection show
```

The host-only lab network used the private subnet `192.168.41.0/24`.

- Kali Linux host-only interface: `192.168.41.7/24`
- Metasploitable 2 target: `192.168.41.8`
- Host-only network: isolated VirtualBox lab network

The NAT interface was used for connectivity where required. Its address and gateway are omitted because they are unnecessary for reproducing the investigations documented here.

Commands used to configure the host-only connection:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname eth1 \
  con-name Lab-HostOnly \
  ipv4.method manual \
  ipv4.addresses 192.168.41.7/24 \
  ipv6.method disabled

sudo nmcli connection up Lab-HostOnly
```

## 2. Initial Network Reconnaissance

Initial service discovery and version detection:

```bash
nmap -sV -sC -oN initial-scan.txt 192.168.41.8
```

The scan identified multiple exposed services on the intentionally vulnerable target. The curated results are documented in [`01-nmap-initial-scan-summary.md`](01-nmap-initial-scan-summary.md).

Raw scan output is not included in this public command log.

## 3. Anonymous FTP Access

Connected to the target's FTP service:

```bash
ftp 192.168.41.8
```

Commands used during the FTP session:

```text
anonymous
pwd
ls -la
bye
```

Anonymous authentication succeeded, but the directory listing did not demonstrate sensitive file disclosure.

See [`../investigations/02-anonymous-ftp-access.md`](../investigations/02-anonymous-ftp-access.md).

## 4. SMB Share Enumeration and Write Testing

Enumerated SMB services and shares:

```bash
nmap -p 139,445 --script smb-os-discovery,smb-enum-shares 192.168.41.8 -oN smb-enumeration.txt
```

Listed the contents of the `tmp` share without supplying credentials:

```bash
smbclient //192.168.41.8/tmp -N -c 'ls'
```

An interactive SMB session was also used to test whether an anonymous user could upload a harmless test file. The test file's original local path and contents are intentionally omitted from this public log.

See [`../investigations/03-samba-anonymous-write-access.md`](../investigations/03-samba-anonymous-write-access.md).

## 5. WebDAV Testing

### Identify HTTP methods

```bash
nmap --script http-methods -p 80 192.168.41.8
```

### Inspect WebDAV capabilities

```bash
/usr/bin/curl -i -X OPTIONS http://192.168.41.8/dav/
```

### Enumerate the WebDAV collection

```bash
/usr/bin/curl -i -X PROPFIND \
  -H "Depth: 1" \
  http://192.168.41.8/dav/
```

### Upload a harmless test file

A locally created test file was uploaded to verify unauthenticated write access:

```bash
/usr/bin/curl -i -T webdav-test.txt \
  http://192.168.41.8/dav/webdav-test.txt
```

### Delete the test file and verify its removal

```bash
/usr/bin/curl -i -X DELETE \
  http://192.168.41.8/dav/webdav-test.txt

/usr/bin/curl -i \
  http://192.168.41.8/dav/webdav-test.txt
```

The upload returned `201 Created`, the deletion returned `204 No Content`, and the follow-up request returned `404 Not Found`.

See [`../investigations/04-webdav-unauthenticated-write.md`](../investigations/04-webdav-unauthenticated-write.md).

## 6. HTTP Server Version Disclosure

Inspected the HTTP response headers:

```bash
/usr/bin/curl -sS -D - -o /dev/null http://192.168.41.8/
```

The response disclosed legacy Apache and PHP version information.

See [`../investigations/05-legacy-web-server-version-disclosure.md`](../investigations/05-legacy-web-server-version-disclosure.md).

## 7. HTTP Security Header Review

Requested the target's HTTP response headers:

```bash
/usr/bin/curl -sS -I http://192.168.41.8/
```

The response was reviewed for the presence of relevant browser security headers.

See [`../investigations/06-missing-http-security-headers.md`](../investigations/06-missing-http-security-headers.md).

## 8. DVWA SQL Injection Investigation

The SQL injection investigation was performed through the DVWA web interface at different security levels.

A single quote was used to test input handling at Low security, and the response was compared with behaviour at High security. The investigation also included reviewing the relevant application source code.

No credentials, session cookies, or authentication tokens are included in this log.

See [`../investigations/07-dvwa-sql-injection.md`](../investigations/07-dvwa-sql-injection.md).

## 9. DVWA Command Injection Investigation

The command injection investigation compared input handling at Low, Medium, and High security levels.

A harmless identity command was used at Low security to verify the execution context of the web application process. Additional inputs were tested at Medium and High security to compare their handling.

The testing did not demonstrate root privilege escalation.

See [`../investigations/08-dvwa-command-injection.md`](../investigations/08-dvwa-command-injection.md).

## 10. Troubleshooting Notes

The following issues were encountered during lab setup:

- `dhclient` was unavailable, so NetworkManager commands were used to manage interfaces.
- Terminal paste issues occasionally introduced unwanted characters into commands.
- A mistyped route command was corrected before continuing.
- `curl` was not always available through the configured shell `PATH`, so `/usr/bin/curl` was used explicitly.

These issues were environmental troubleshooting observations rather than security findings.

## 11. Evidence and Sanitisation

This public command log is a curated record rather than a complete terminal history.

The following have been excluded or minimised:

- NAT interface addresses and gateway details.
- Unnecessary IPv6 and interface metadata.
- MAC addresses and SSH host-key fingerprints.
- Database metadata and other unrelated service fingerprints.
- Credentials, session identifiers, and authentication material.
- Potentially identifying local file contents.
- Unreviewed raw scan output.

Private host-only addresses are retained only where they help explain the isolated lab setup or reproduce a documented finding.

All findings should be interpreted within the scope of the authorised training environment. Potential impact is distinguished from behaviour that was directly demonstrated.
