# Investigation 02 — Anonymous FTP Access

## 1. Objective

Determine whether the FTP service permits unauthenticated access and whether accessible files are exposed.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM
- **Target IP:** `192.168.41.8`
- **Port:** TCP 21
- **Environment:** Authorised, isolated VirtualBox host-only lab

## 3. Methodology

Connected to the target FTP service and attempted to authenticate using the anonymous account. After successful authentication, the working directory and directory listing were reviewed.

No files were downloaded, modified, or deleted during this test.

## 4. Results

The server accepted anonymous authentication and returned a successful login response.

The directory listing contained only the current directory (`.`) and parent directory (`..`). No additional files were observed during this test.

**Confirmed behaviour:** Unauthenticated users can establish an FTP session.

**Not demonstrated:** Access to sensitive files, modification of server files, or further compromise.

## 5. Security Impact

Anonymous FTP access can expose files to unauthenticated users if the service is configured to make files available. The practical impact depends on the permissions and contents of the accessible directories.

In this test, anonymous access was confirmed, but no sensitive information disclosure was demonstrated. The finding should therefore be treated as **Low or Informational**, depending on whether anonymous FTP is required by the intended service design.

## 6. Evidence

Evidence should be added only after the screenshot has been reviewed for sensitive information.

Planned evidence: a sanitised screenshot showing the successful anonymous login and the limited directory listing.

## 7. Recommendations

- Disable anonymous FTP if it is not explicitly required.
- If anonymous access is required, restrict it to a dedicated directory.
- Apply read-only permissions unless write access is essential.
- Avoid storing sensitive files within anonymously accessible locations.
- Prefer a secure, authenticated transfer method such as SFTP where appropriate.

## 8. Retest Criteria

After remediation, repeat the anonymous login attempt.

- **Pass:** Anonymous authentication is rejected, or access is restricted to the explicitly approved directory and permissions.
- **Fail:** Anonymous users can access unintended files or perform unauthorised actions.

## 9. Limitations

This test verified authentication behaviour and the visible directory listing only. It did not assess every FTP command or directory permission.

