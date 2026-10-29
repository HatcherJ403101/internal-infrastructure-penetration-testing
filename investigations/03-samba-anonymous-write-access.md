# Investigation 03 — Samba Anonymous Write Access

## 1. Objective

Assess whether the target's SMB service exposes shares to unauthenticated users and whether anonymous access permits file creation.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM
- **Target IP:** `192.168.41.8`
- **Ports:** TCP 139 and 445
- **Environment:** Authorised, isolated VirtualBox host-only lab

## 3. Methodology

SMB share enumeration was performed using Nmap scripts:

`nmap -p 139,445 --script smb-os-discovery,smb-enum-shares 192.168.41.8 -oN smb-enumeration.txt`

The `tmp` share was then accessed using the SMB client without supplying credentials. A controlled test file was uploaded to verify whether anonymous write access was permitted.

## 4. Results

Share enumeration identified the `tmp` share as accessible for anonymous read and write operations. Anonymous access to other enumerated shares was not uniformly permitted.

The following behaviour was confirmed:

- The `tmp` share could be accessed without credentials.
- A directory listing could be retrieved.
- A test file could be uploaded without authentication.

The test file was created to verify write permissions. No code execution or access to sensitive data was demonstrated.

## 5. Security Impact

Unauthenticated write access to a network share can allow an unauthorised user to place, replace, or disrupt files within the share. Depending on how the files are used by other services or users, this could contribute to further compromise.

**Assessment:** High potential risk in an environment where the share is accessible to untrusted systems. The rating should be reconsidered in the context of the intended use, network exposure, and effective permissions.

The test confirmed unauthenticated file creation; it did not establish remote code execution.

## 6. Evidence

Evidence should be uploaded only after sanitisation.

Planned evidence:

- Sanitised SMB share enumeration showing the anonymous access permissions.
- Sanitised terminal evidence showing access to the `tmp` share and successful test-file creation.

Do not publish the uploaded test file's contents if they contain local system information.

## 7. Recommendations

- Disable anonymous access to SMB shares unless explicitly required.
- Remove anonymous write permissions from shared directories.
- Apply least-privilege access controls to each share.
- Restrict SMB connectivity to trusted systems and required network segments.
- Upgrade unsupported Samba software to a supported release.
- Review the share for unauthorised or unexpected files.

## 8. Retest Criteria

Repeat share enumeration using an unauthenticated account and attempt a controlled write test after remediation.

- **Pass:** Anonymous users cannot write to the share, and any permitted anonymous read access is explicitly approved and restricted.
- **Fail:** An unauthenticated user can still create or modify files where this is not intended.

## 9. Limitations

This investigation verified anonymous access and file creation in the `tmp` share. It did not test code execution, privilege escalation, or the security of every SMB share.

