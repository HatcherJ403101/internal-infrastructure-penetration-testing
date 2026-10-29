# Investigation 08 — DVWA Operating-System Command Injection

## 1. Objective

Assess how the DVWA Command Injection module handles user-supplied input at different security levels and determine whether the input can cause unintended operating-system commands to execute.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM running DVWA
- **Target address:** `192.168.41.8` (private lab network)
- **Application:** DVWA Command Injection module
- **Environment:** Authorised, isolated VirtualBox host-only lab

Testing was limited to the intentionally vulnerable application.

## 3. Methodology

The module was first tested using a normal IP address to confirm the expected ping functionality.

At Low security, a controlled command-separator test was used to determine whether additional operating-system commands could be executed. The behaviour was then compared with the Medium and High security levels.

The test used the `id` command to identify the effective account running the executed command. No attempt was made to escalate privileges.

## 4. Results

### Low security

A normal IP address produced the expected ping output.

The input:

`192.168.41.8; id`

produced the ping results followed by:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This confirms that the application executed the additional operating-system command. The output identified the effective account as `www-data`.

The reviewed Low-level source code concatenated the supplied input into a command passed to `shell_exec()`, allowing the shell command separator to be interpreted.

### Medium security

The reviewed source code attempted to remove the `&&` and `;` character sequences before constructing the command.

The tested semicolon and `&&` inputs did not produce evidence of successful additional command execution. These tests do not establish that every possible bypass is prevented.

### High security

The reviewed source code split the supplied value into four dot-separated components and checked that each was numeric before rebuilding the IP address.

A valid IP address produced the expected ping output. The tested input:

`192.168.41.8; id`

was rejected with an invalid-IP message.

These results demonstrate that the tested input was rejected at High security. They are not proof that the entire application is free from command injection.

## 5. Security Impact

Command injection can allow user-controlled input to execute unintended operating-system commands with the privileges of the application process.

In this test, command execution was confirmed under `www-data`. No root access, privilege escalation, persistence, or broader host compromise was demonstrated.

**Assessment:** High potential severity in an exposed application, subject to the deployment context and the privileges and reachability of the affected process.

## 6. Evidence

Evidence should be reviewed and sanitised before publication.

Planned evidence:

- A screenshot showing the Low-level input and `www-data` command output.
- A screenshot showing the tested Medium-level input and its observed result.
- A screenshot showing High-level rejection of the invalid input.
- A screenshot of normal ping behaviour at High security, if useful for comparison.

Ensure screenshots do not expose credentials, session cookies, session identifiers, or unrelated system information.

## 7. Recommendations

- Avoid passing user-controlled input into shell command strings.
- Prefer APIs that execute a specific program with arguments supplied separately, without invoking a shell.
- Validate IP addresses using a reliable IP-address parser rather than relying solely on string checks.
- Run application services with the minimum privileges required.
- Return controlled error messages and avoid exposing unnecessary command output.
- Review related input-handling code paths and retest after remediation.

## 8. Retest Criteria

Repeat the normal ping test and the controlled command-injection test after remediation.

- **Pass:** Valid input produces the intended result, while crafted input cannot cause additional commands to execute.
- **Fail:** User-controlled input can still alter the command executed by the application.

Testing should also verify that the application process retains only the privileges it needs.

## 9. Limitations

This investigation confirmed command injection at Low security and compared a limited set of inputs at Medium and High security. It did not test all possible payloads, demonstrate privilege escalation, or establish that the higher security levels are comprehensively secure.


