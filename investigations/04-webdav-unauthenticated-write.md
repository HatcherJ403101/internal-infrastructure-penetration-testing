# Investigation 04 — Unauthenticated WebDAV Write Access

## 1. Objective

Determine whether the target's WebDAV endpoint permits unauthenticated users to create files and identify the potential security implications.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM
- **Target IP:** `192.168.41.8`
- **Endpoint:** `/dav/`
- **Port:** TCP 80 (HTTP)
- **Environment:** Authorised, isolated VirtualBox host-only lab

## 3. Methodology

The WebDAV endpoint was inspected using HTTP requests to identify its supported methods and behaviour.

The following requests were used:

- `OPTIONS` to inspect the methods advertised by the endpoint.
- `PROPFIND` to inspect the directory contents.
- An HTTP `PUT` request to upload a harmless test file.
- An HTTP `DELETE` request to remove the test file.
- A subsequent request to confirm that the test file was no longer accessible.

All testing was performed against the designated training VM.

## 4. Results

The endpoint responded with `DAV: 1,2` and advertised methods including `PUT`-related WebDAV functionality, file deletion, and directory operations.

An unauthenticated `PUT` request successfully created a test file and returned **HTTP 201 Created**.

The test file was then deleted successfully, with the server returning **HTTP 204 No Content**. A subsequent request returned **HTTP 404**, consistent with the file no longer being available.

The directory listing was also enabled. During this test, it showed no files other than the parent-directory entry.

**Confirmed behaviour:** An unauthenticated user could create a file in the WebDAV directory.

**Not demonstrated:** Execution of uploaded content, access to sensitive files, or compromise of the host.

## 5. Security Impact

Unauthenticated file creation can permit unauthorised content placement and may create further risks if uploaded files can be accessed or processed by other services. The practical impact depends on server configuration, filesystem permissions, and how the uploaded content is handled.

The confirmed write capability warrants remediation where anonymous uploads are not an intended requirement. The test does not establish remote code execution.

## 6. Evidence

Evidence should be added only after reviewing and sanitising the screenshots.

Planned evidence:

- A sanitised HTTP response showing the successful test upload.
- A sanitised response showing successful deletion.
- Optional directory-listing evidence, if useful to demonstrate the observed configuration.

Do not publish session cookies, authentication tokens, or unrelated browser information.

## 7. Recommendations

- Disable WebDAV if it is not required.
- Require authentication and enforce appropriate authorisation for write operations.
- Restrict write permissions to the minimum necessary locations.
- Disable unnecessary WebDAV methods.
- Disable directory indexing where directory listings are not intended.
- Upgrade unsupported Apache and related components.
- Restrict access to the service to trusted network segments where appropriate.

## 8. Retest Criteria

After remediation, repeat the unauthenticated upload attempt using a harmless test file.

- **Pass:** Unauthenticated file creation is rejected, or access is restricted to an explicitly approved and appropriately protected workflow.
- **Fail:** Unauthenticated users can still create files where this is not intended.

If directory indexing is disabled, verify that a directory request no longer exposes an index listing.

## 9. Limitations

This test verified unauthenticated file creation and deletion in the designated WebDAV directory. It did not test whether uploaded files could be executed, whether other directories were writable, or whether the host could be compromised through WebDAV.

