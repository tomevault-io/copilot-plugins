## ios-security-auditor

> Security audit for iOS projects (OWASP Mobile Top 10 2024)


# iOS Security Auditor

When working with iOS code, check for OWASP Mobile Top 10 2024 vulnerabilities.

## Data Storage (M9)

Flag insecure storage:
- Sensitive data in UserDefaults (tokens, passwords, PII, API keys)
- Sensitive data in plist files or bundle resources
- Missing Keychain usage for credentials
- Incorrect kSecAttrAccessible flags (avoid kSecAttrAccessibleAlways)
- Unprotected files in Documents/tmp without .fileProtection
- Unencrypted Core Data stores with sensitive data

## Hardcoded Secrets (M1)

Scan for exposed credentials:
- API keys hardcoded in source
- Client secrets, tokens, passwords in code
- Private keys or certificates in bundle
- Database connection strings
- Secrets in Info.plist or xcconfig files
- Test credentials persisting to release builds

## Network Security (M5)

Check network configuration:
- NSAllowsArbitraryLoads disabled (ATS enabled)
- No overly permissive exception domains
- TLS 1.2+ enforcement
- No ServerTrustPolicy.disableEvaluation
- No allowsInvalidSSLCertificate flags
- Sensitive data not in URL query parameters

## Authentication (M3)

Review auth implementation:
- Token storage uses Keychain
- Server-side authorization for protected resources
- Session invalidation on logout
- No client-side only auth checks

## Input Validation (M4)

Check for injection vulnerabilities:
- Core Data predicates using parameterized queries
- WKWebView evaluateJavaScript sanitizes input
- URL scheme handlers validate parameters
- File operations validate paths (no ../ traversal)

## Output

Flag issues by severity:
- CRITICAL: Immediate fix required (cite OWASP Mxx)
- HIGH/MEDIUM/LOW: With remediation steps

---
> Source: [gygantskiyMatilyock/ios-developer-agents](https://github.com/gygantskiyMatilyock/ios-developer-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
