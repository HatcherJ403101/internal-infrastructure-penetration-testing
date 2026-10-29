# Investigation 06 — Missing HTTP Security Headers

## 1. Objective

Review the target's HTTP response headers for common browser security headers and assess whether additional browser-side protections may be appropriate.

## 2. Scope and Environment

- **Target:** Metasploitable 2 training VM
- **Target IP:** `192.168.41.8`
- **Endpoint:** `/`
- **Port:** TCP 80 (HTTP)
- **Environment:** Authorised, isolated VirtualBox host-only lab

## 3. Methodology

The response headers were inspected using:

`curl -sS -I http://192.168.41.8/`

The returned headers were reviewed for common browser security controls.

## 4. Results

The server returned HTTP `200 OK`. The observed response did not include the following headers:

- `Content-Security-Policy`
- `X-Frame-Options`
- `X-Content-Type-Options`

The response was served over HTTP rather than HTTPS. As a result, this test does not establish whether HTTP Strict Transport Security (HSTS) is configured on a separate HTTPS endpoint.

## 5. Security Impact

The impact of missing security headers depends on the application's functionality and other existing protections.

- **Content-Security-Policy (CSP)** can limit the sources from which browsers load content and may reduce the impact of certain content-injection attacks.
- **X-Frame-Options**, or an appropriate CSP `frame-ancestors` directive, can help control whether pages may be embedded in frames.
- **X-Content-Type-Options: nosniff** can prevent certain forms of browser MIME-type sniffing.

Their absence does not independently demonstrate a successful attack. This finding is best treated as **Informational or Low** pending application-specific assessment.

## 6. Evidence

Evidence should be added only after review and sanitisation.

Planned evidence: a sanitised response-header capture showing the headers returned by the server. Avoid including cookies, session identifiers, or unrelated response data.

## 7. Recommendations

- Define a Content Security Policy appropriate to the application's resources and functionality.
- Configure `X-Content-Type-Options: nosniff` where appropriate.
- Use CSP `frame-ancestors` or `X-Frame-Options` to restrict unwanted framing.
- Deploy HTTPS and assess HSTS on the HTTPS endpoint.
- Test proposed header policies against the application to avoid breaking required functionality.

## 8. Retest Criteria

Repeat the header inspection after configuration changes.

- **Pass:** Appropriate headers are returned and their policies match the application's security requirements.
- **Fail:** Required headers remain absent or are configured ineffectively.

## 9. Limitations

This was a review of one HTTP endpoint's response headers. It did not test for clickjacking, content injection, or other browser-based attacks, and it did not assess an HTTPS endpoint.

