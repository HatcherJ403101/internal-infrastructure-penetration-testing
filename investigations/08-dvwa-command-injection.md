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

The module was tested with a normal IP address to establish the expected ping behaviour.

At Low security, a controlled command-separator test was used to determine whether additional operating-system commands could execute. Tests were then conducted at Medium and High security to compare the observed behaviour.

The `id` command was used to identify the effective account under which the additional command executed. No attempt was made to escalate privileges.

The investigation also included reviewing the relevant DVWA source code at each security level.

## 4. Results

### Low security

A normal IP address produced the expected ping output.

The following input was tested:

`192.168.41.8; id`

The response contained the ping output and statistics, followed by:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This confirms that the application executed the additional operating-system command. The output identifies the effective user as `www-data`, rather than root.

The reviewed Low-security source code concatenated the supplied input into a command passed to `shell_exec()`. This allowed the shell command separator to be interpreted, enabling the additional command to execute.

**Result:** Command injection was demonstrated at Low security, with command execution under the web application's `www-data` account.

### Medium security

A normal IP address, `192.168.41.8`, produced the expected ping output, including three replies and ping statistics. This established that normal ping functionality worked while Medium security was selected.

Two additional inputs were tested:

`192.168.41.8; id`

`192.168.41.8 && id`

For both tests, the form returned to its previous state and no visible response appeared below the input field. Neither test displayed the `uid`, `gid`, or `www-data` output observed during the Low-security test.

The reviewed Medium-security source code attempted to remove the `&&` and `;` character sequences before constructing the command. The observed behaviour is consistent with the input being altered by this filtering, but the blank responses alone do not establish why no output appeared or prove that every command-injection route is prevented.

**Result:** The tested inputs did not produce visible evidence of additional command execution at Medium security. This is an observation limited to the inputs tested, not proof of comprehensive protection.

### High security

The reviewed High-security source code split the supplied value into four dot-separated components and checked that each component was numeric before rebuilding the IP address.

A valid IP address produced the expected ping output. The following input was also tested:

`192.168.41.8; id`

The application rejected this input with an invalid-IP message.

These results demonstrate that the tested input was rejected at High security. They do not prove that every possible command-injection path is prevented.

**Result:** The tested input was rejected at High security, while normal ping functionality continued to work.

## 5. Security Impact

Command injection occurs when user-controlled input can alter a command executed by an operating system. Depending on the application's privileges and environment, this can allow unintended commands to run and potentially expose data or enable further compromise.

In this investigation, additional command execution was confirmed under the `www-data` account at Low security.

The testing did not demonstrate root access, privilege escalation, persistence, or broader host compromise.

**Potential severity:** High where an exposed application allows unauthorised command execution and the affected process has access to sensitive resources. Actual risk depends on deployment context, exposure, process privileges, and accessible resources.

## 6. Evidence

Evidence was reviewed and sanitised before publication. The screenshots should be interpreted alongside the methodology and observations in this report.

### Low-security command execution

**Evidence file:** `08-command-injection-low-id.png`

The screenshot shows the input `192.168.41.8; id`, the ping output, and the subsequent identity information:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This supports the finding that the additional command executed in the web application's process context.

**Screenshot limitation:** The screenshot does not visibly display the active DVWA security level. The test was performed with the security level set to Low, as recorded in the investigation procedure and test notes. The image itself does not independently establish the selected level and must not be treated as standalone visual proof of that setting.

### Medium-security configuration and baseline

**Evidence file:** `08-command-injection-medium-security-setting.png`

This screenshot documents the Medium security level selected in DVWA. It establishes the configuration shown at the time the screenshot was taken, but does not independently prove which level was active during each individual test.

**Evidence file:** `08-command-injection-medium-normal-ping.png`

This screenshot shows a normal ping to `192.168.41.8` working as expected at Medium security. It provides a baseline for comparing the behaviour of the additional command-injection tests.

The semicolon and `&&` tests at Medium returned no visible output. No screenshot is relied upon as standalone proof that these inputs were blocked, because the form returned to its previous state and the submitted input was no longer visible alongside a diagnostic response.

### High-security comparison

The High-security tests compared a normal IP address with the input `192.168.41.8; id`. The normal input produced the expected ping behaviour, while the crafted input was rejected with an invalid-IP message.

The corresponding screenshots, if published, should use these filenames:

- `08-command-injection-high-invalid-ip.png`
- `08-command-injection-high-normal-ping.png`

The results are limited to the inputs tested and do not establish that all possible bypasses are prevented.

### Evidence handling

Screenshots must not expose credentials, session cookies, session identifiers, authentication tokens, or unrelated system information. Command output must remain unaltered; sanitisation should be limited to irrelevant or sensitive surrounding information.

## 7. Recommendations

- Avoid passing user-controlled input into shell command strings.
- Prefer APIs that execute a specific program with arguments supplied separately, without invoking a shell.
- Validate IP addresses with a reliable IP-address parser rather than relying solely on manual string checks.
- Run application services with the minimum privileges required.
- Return controlled error messages and avoid exposing unnecessary command output.
- Review related input-handling code paths and retest after remediation.

## 8. Retest Criteria

Repeat the normal ping test and the controlled command-injection tests after remediation.

- **Pass:** Valid input produces the intended result, while crafted input cannot cause additional unintended commands to execute.
- **Fail:** User-controlled input can still alter the command executed by the application.

Testing should also verify that the application process retains only the privileges necessary for its function.

## 9. Limitations

This investigation confirmed command injection at Low security and compared a limited set of inputs at Medium and High security.

The Medium tests returned no visible output for the tested semicolon and `&&` inputs. This observation does not, on its own, prove that the inputs were safely blocked.

The Low-security screenshot does not display the selected security level, and the Medium configuration screenshot documents the setting at the time it was captured rather than proving the setting during every test.

The investigation did not test every possible payload, demonstrate privilege escalation, or establish that the higher security levels are comprehensively secure. Findings are limited to the behaviour observed in the isolated DVWA training environment.

The investigation did not test every possible payload, demonstrate privilege escalation, or establish that the higher security levels are comprehensively secure. Findings are limited to the behaviour observed in the isolated DVWA training environment.
