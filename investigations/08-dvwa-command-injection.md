# Investigation 08 — DVWA Operating-System Command Injection

## 1. Objective

Assess how the DVWA Command Injection module handles user-supplied input at different security levels and determine whether that input can cause unintended operating-system commands to execute.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM running DVWA
- **Target address:** `192.168.41.8` (private lab network)
- **Application:** DVWA Command Injection module
- **Environment:** Authorised, isolated VirtualBox host-only lab

Testing was limited to the intentionally vulnerable application within the isolated training environment.

## 3. Methodology

The module was first tested with a normal IP address to confirm the expected ping functionality.

At Low security, a controlled command-separator test was used to determine whether additional operating-system commands could be executed. The observed behaviour was then compared with tests at the Medium and High security levels.

The `id` command was used to identify the effective account under which the additional command executed. No attempt was made to escalate privileges.

The investigation also included reviewing the relevant DVWA source code at each security level to understand how user-supplied input was processed.

## 4. Results

### Low security

A normal IP address produced the expected ping output.

The following input was then tested:

`192.168.41.8; id`

The response contained the ping output and statistics, followed by:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This confirms that the application executed the additional operating-system command. The output identifies the effective user as `www-data`, rather than root.

The reviewed Low-security source code concatenated the supplied input into a command passed to `shell_exec()`. This allowed the shell command separator to be interpreted, enabling the additional command to execute.

**Result:** Command injection was demonstrated at Low security, with command execution under the web application's `www-data` account.

### Medium security

The reviewed Medium-security source code attempted to remove the `&&` and `;` character sequences before constructing the command.

The tested semicolon and `&&` inputs did not produce evidence of successful additional command execution.

These results are limited to the inputs tested. They do not establish that every possible bypass is prevented or that the application is secure against command injection.

**Result:** The tested inputs did not demonstrate additional command execution at Medium security.

### High security

The reviewed High-security source code split the supplied value into four dot-separated components and checked that each component was numeric before rebuilding the IP address.

A valid IP address produced the expected ping output. The following input was also tested:

`192.168.41.8; id`

The application rejected this input with an invalid-IP message.

These results demonstrate that the tested input was rejected at High security. They do not prove that all possible command-injection paths are prevented.

**Result:** The tested input was rejected at High security, while normal ping functionality continued to work.

## 5. Security Impact

Command injection occurs when user-controlled input can alter a command executed by an operating system. Depending on the application's privileges and environment, this can allow unintended commands to run, potentially exposing data or enabling further compromise.

In this investigation, additional command execution was confirmed under the `www-data` account.

The testing did **not** demonstrate root access, privilege escalation, persistence, or broader host compromise. The results were obtained in an intentionally vulnerable training environment and should not be interpreted as evidence that a production system was compromised.

**Potential severity:** High where an exposed application allows unauthorised command execution and the affected process has access to sensitive resources. Actual risk depends on deployment context, exposure, process privileges, and accessible resources.

## 6. Evidence

Evidence was reviewed and sanitised before publication.

### Low-security command execution

**Evidence file:** `08-command-injection-low-id.png`

The screenshot shows the tested input `192.168.41.8; id`, the ping output, and the subsequent identity information:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This output supports the finding that the additional `id` command executed in the web application's process context.

**Screenshot limitation:** The screenshot does not visibly display the active DVWA security level. The test was performed with the security level set to **Low**, as recorded in the investigation procedure and test notes. The image itself does not independently establish that setting. It must therefore be interpreted alongside the written methodology and source-code analysis, rather than as standalone visual proof of the selected security level.

The output identifies the command's effective account as `www-data`. It does not demonstrate root access or privilege escalation.

### Medium-security comparison

The Medium-security tests compared semicolon and `&&` inputs with the application's observed responses. The tested inputs did not demonstrate successful additional command execution.

### High-security comparison

The High-security tests compared a normal IP address with the tested input containing `; id`. The normal input produced the expected ping behaviour, while the crafted input was rejected with an invalid-IP message.

The Medium- and High-security observations are limited to the specific inputs tested and should not be treated as proof that all potential bypasses are prevented.

### Evidence handling

Screenshots must not expose credentials, session cookies, session identifiers, authentication tokens, or unrelated system information. The recorded command output should remain unaltered; any necessary sanitisation should be limited to irrelevant or sensitive surrounding information.

## 7. Recommendations

- Avoid passing user-controlled input into shell command strings.
- Prefer APIs that execute a specific program with arguments supplied separately, without invoking a shell.
- Validate IP addresses with a reliable IP-address parser rather than relying solely on manual string checks.
- Run application services with the minimum privileges required.
- Return controlled error messages and avoid exposing unnecessary command output.
- Review related input-handling code paths and retest after remediation.

## 8. Retest Criteria

Repeat the normal ping test and controlled command-injection tests after remediation.

- **Pass:** Valid input produces the intended result, while crafted input cannot cause additional unintended commands to execute.
- **Fail:** User-controlled input can still alter the command executed by the application.

Testing should also verify that the application process retains only the privileges necessary for its function.

## 9. Limitations

This investigation confirmed command injection at Low security and compared a limited set of inputs at Medium and High security.

The screenshot of the Low-security result does not display the selected security level; that context is documented in this report and should be read alongside the evidence.

The investigation did not test every possible payload, demonstrate privilege escalation, or establish that the higher security levels are comprehensively secure. Findings are limited to the behaviour observed in the isolated DVWA training environment.
