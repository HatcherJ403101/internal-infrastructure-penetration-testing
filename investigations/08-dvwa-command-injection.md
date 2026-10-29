# Investigation 08 — DVWA Operating-System Command Injection

## 1. Objective

Assess how the DVWA Command Injection module handles user-supplied input at different security levels and determine whether that input can cause unintended operating-system commands to execute.

The investigation compares the observed behaviour at Low, Medium, and High security, supported by controlled testing, source-code review, and screenshots of the results.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM running DVWA
- **Target address:** `192.168.41.8` (private lab network)
- **Application:** DVWA Command Injection module
- **Environment:** Authorised, isolated VirtualBox host-only lab
- **Testing scope:** Command injection behaviour within the intentionally vulnerable DVWA application

Testing was confined to the isolated training environment. No external systems were targeted.

## 3. Methodology

The investigation used the following procedure:

1. Submit a valid IP address to confirm that the expected ping functionality works.
2. At Low security, use a controlled command-separator input to test whether an additional operating-system command can execute.
3. At Medium security, confirm the selected security level, establish a normal-ping baseline, and test semicolon and `&&` inputs.
4. At High security, confirm the selected security level, establish normal ping behaviour, and test whether a crafted input is rejected.
5. Review the relevant DVWA source code to understand how input is handled at each security level.
6. Record the observed results and distinguish confirmed command execution from behaviour that remains inconclusive.

The `id` command was used to identify the effective account running the additional command. No attempt was made to escalate privileges.

## 4. Results

### 4.1 Low security

A normal IP address produced the expected ping output.

The following input was then submitted:

`192.168.41.8; id`

The response contained the ping output and statistics, followed by:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This confirms that the application executed the additional operating-system command. The output identifies the effective user as `www-data`, rather than root.

The reviewed Low-security source code concatenated the supplied input into a command passed to `shell_exec()`. This allowed the shell to interpret the semicolon as a command separator, enabling the additional command to execute.

**Finding:** Command injection was demonstrated at Low security, with command execution under the web application's `www-data` account.

**Evidence limitation:** The screenshot captures the input and command output but does not visibly display the active DVWA security level. The test was performed with Low selected, as recorded in the investigation notes. The screenshot should therefore be read alongside this report rather than treated as standalone visual proof of the selected security level.

### 4.2 Medium security

The DVWA security setting was changed to Medium and confirmed on the DVWA Security page before the Medium tests were performed.

A normal ping using the input:

`192.168.41.8`

returned three replies and ping statistics. This established that normal ping functionality worked with Medium selected.

Two additional inputs were tested.

**Semicolon test:**

`192.168.41.8; id`

**Ampersand test:**

`192.168.41.8 && id`

For both tests, the form returned to its previous state and no visible response appeared below the input field. Neither test displayed the `uid`, `gid`, or `www-data` output observed during the Low-security test.

The reviewed Medium-security source code attempted to remove the `&&` and `;` character sequences before constructing the command. The observed behaviour is consistent with the input being altered by this filtering.

However, the blank responses do not independently establish why no output appeared. They also do not prove that every possible command-injection route is prevented.

**Finding:** The tested semicolon and `&&` inputs did not produce visible evidence of additional command execution at Medium security. This is a limited observation of the tested inputs, not proof of comprehensive protection.

### 4.3 High security

The DVWA security setting was changed to High and confirmed on the DVWA Security page before the High-security tests were performed.

A valid IP address produced the expected ping output.

The following crafted input was then tested:

`192.168.41.8; id`

The application rejected this input with an invalid-IP message.

The reviewed High-security source code split the supplied value into four dot-separated components and checked that each component was numeric before rebuilding the IP address.

The rejection is consistent with this input-validation behaviour. The normal-ping result also confirms that the tested valid address continued to work.

**Finding:** The tested crafted input was rejected at High security, while normal ping functionality worked. This does not prove that every possible command-injection path is prevented.

## 5. Security Impact

Operating-system command injection occurs when user-controlled input can alter a command executed by the operating system. Depending on the application's exposure, privileges, and accessible resources, this can lead to unauthorised command execution, data exposure, or further compromise.

In this investigation, additional command execution was confirmed under the `www-data` account at Low security.

The investigation did **not** demonstrate:

- Root access or privilege escalation.
- Persistence on the target.
- Access to other users' data.
- Broader host compromise.
- Successful additional command execution in the tested Medium-security cases.

The Medium and High results are limited to the inputs tested. They should not be interpreted as proof that those security levels are comprehensively secure.

**Potential severity:** High where an exposed application permits unauthorised command execution and the affected process can access sensitive resources. Actual risk depends on the deployment context, exposure, process privileges, and accessible resources.

## 6. Evidence

The screenshots below are intended to support the findings described in this report. Each image has a specific purpose; the evidence should be interpreted alongside the methodology, source-code review, results, and limitations.

### 6.1 Low-security command execution

**File:** [`08-command-injection-low-id.png`](../evidence/08-command-injection-low-id.png)

The screenshot shows the input `192.168.41.8; id`, the ping output, and the subsequent identity information:

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

This supports the finding that the additional operating-system command executed in the web application's process context.

**Limitation:** The screenshot does not visibly display the active DVWA security level. The test was performed with Low selected, as recorded in the investigation notes, but that setting cannot be independently verified from this image alone.

### 6.2 Medium-security configuration

**File:** [`08-command-injection-medium-security-setting.png`](../evidence/08-command-injection-medium-security-setting.png)

This screenshot shows Medium selected on the DVWA Security page. It documents the selected configuration when the screenshot was captured and supports the test context.

The screenshot does not independently prove which security setting was active during every individual command submission. The test sequence and observations are recorded in Section 4.2.

### 6.3 Medium-security normal-ping baseline

**File:** [`08-command-injection-medium-normal-ping.png`](../evidence/08-command-injection-medium-normal-ping.png)

This screenshot shows a normal ping to `192.168.41.8` returning three replies and ping statistics while Medium security was selected.

It establishes that the normal application function worked and provides a baseline for comparison with the additional command-injection tests.

### 6.4 High-security configuration

**File:** [`08-command-injection-high-security-setting.png`](../evidence/08-command-injection-high-security-setting.png)

This screenshot shows High selected on the DVWA Security page. It documents the selected configuration when the screenshot was captured and supports the context for the High-security tests.

As with the Medium configuration screenshot, it does not independently prove the active setting during every individual command submission. The test sequence and observations are recorded in Section 4.3.

### 6.5 High-security normal-ping baseline

**File:** [`08-command-injection-high-normal-ping.png`](../evidence/08-command-injection-high-normal-ping.png)

This screenshot shows a normal ping using the valid target IP address. It provides a comparison against the crafted input tested at High security.

### 6.6 High-security invalid-input result

**File:** [`08-command-injection-high-invalid-ip.png`](../evidence/08-command-injection-high-invalid-ip.png)

This screenshot shows the input `192.168.41.8; id` being rejected with an invalid-IP message.

The result supports the observation that this particular crafted input was rejected at High security. It does not establish that every possible command-injection attempt would be rejected.

### 6.7 Evidence handling

Screenshots were reviewed and sanitised before publication. Credentials, session cookies, session identifiers, authentication tokens, and unrelated system information must not be exposed.

The observed command output must remain unaltered. Sanitisation should be limited to irrelevant or sensitive surrounding information.

The Medium tests returned no visible response for the tested semicolon and `&&` inputs. Those blank results are documented in this report but are not presented as standalone proof that the inputs were safely blocked.

## 7. Recommendations

- **Avoid shell command construction:** Do not concatenate user-controlled input into operating-system command strings.
- **Use safer execution interfaces:** Prefer APIs that execute a specific program with arguments supplied separately, without invoking a shell.
- **Validate input reliably:** Use a suitable IP-address parser rather than relying solely on manual character filtering or string checks.
- **Apply least privilege:** Run application services with only the privileges necessary for their intended function.
- **Control command output:** Avoid exposing unnecessary operating-system output or internal error details to users.
- **Review related code paths:** Examine other inputs and functions that construct or execute operating-system commands.
- **Retest after remediation:** Verify that valid input continues to work and crafted input cannot cause unintended additional commands to execute.

## 8. Retest Criteria

Repeat the normal ping and controlled command-injection tests after remediation.

- **Pass:** Valid IP input produces the expected result, while crafted input cannot cause additional unintended operating-system commands to execute.
- **Fail:** User-controlled input can alter the command executed by the application.
- **Additional verification:** Confirm that the application process retains only the privileges required for its function and that errors do not disclose unnecessary internal details.

A successful limited set of tests should not be treated as proof that all command-injection paths have been eliminated. Retesting should cover the relevant input-handling code paths.

## 9. Limitations

This investigation confirmed command injection at Low security and compared a limited set of inputs at Medium and High security.

The Medium semicolon and `&&` tests returned no visible response. Although this behaviour is consistent with the reviewed filtering logic, the blank responses alone do not establish why the application displayed no output or prove that all command-injection routes are blocked.

The High-security test rejected the specific crafted input tested. It does not prove comprehensive protection against all potential inputs.

The Low-security screenshot does not display the selected security level. The separate Medium- and High-security setting screenshots document the configuration when captured but do not independently establish the active setting during every individual command submission.

No privilege escalation, root access, persistence, or broader host compromise was demonstrated. Findings are limited to the behaviour observed in the authorised, isolated DVWA training environment.
