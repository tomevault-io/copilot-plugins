## armory

> OWASP Top 10 vulnerability scanner producing severity-ranked findings with


# Security Reviewer

OWASP Top 10 vulnerability scanner producing severity-ranked findings with
exploit scenarios, impact assessment, and remediation code.

---

## Scope and Trigger Conditions

Activate when:

- User requests security review, vulnerability scan, or security audit
- User asks to check for injection, XSS, auth issues, or OWASP patterns
- Code handles user input, authentication, database queries, or file operations

Do NOT activate when:

- User asks for general code quality review (use code-reviewer)
- User asks for secret/credential detection only (use secret-scanner)
- User asks for dependency vulnerability audit (use dependency-audit)
- User asks for repository-level security posture (use repo-sentinel)

---

## Analysis Phases

### Phase 1: Attack Surface Mapping

1. Identify entry points:
   - HTTP route handlers, API endpoints, GraphQL resolvers
   - CLI argument parsers, stdin readers
   - File upload handlers, webhook receivers
   - Message queue consumers, event handlers
2. Map data flow from entry points to:
   - Database queries (SQL, ORM, NoSQL)
   - File system operations
   - External HTTP requests
   - Template rendering
   - Command execution
   - Serialization/deserialization
3. Classify each data flow by trust boundary crossings

### Phase 2: Injection Analysis

Scan for injection vectors across all identified data flows:

**SQL Injection**

- String concatenation or f-strings in SQL queries
- Dynamic table/column names from user input
- ORM raw query usage without parameterization
- Stored procedure calls with unsanitized input
- Flag: any user input reaching SQL without parameterized queries

**Command Injection**

- `os.system()`, `subprocess.run(shell=True)`, `exec()`, `eval()`
- Template strings in shell commands
- User input in `child_process.exec()` (Node.js)
- Flag: any external input reaching command execution

**NoSQL Injection**

- MongoDB query operators from user input (`$gt`, `$ne`, `$where`)
- Unvalidated JSON in query construction
- Flag: user-controlled query operators

**Template Injection**

- User input in template strings (Jinja2, Handlebars, EJS)
- Server-side template injection via render context
- Flag: user input reaching template engine without escaping

### Phase 3: Cross-Site Scripting (XSS)

Scan for XSS vectors:

**Reflected XSS**

- User input rendered in HTML responses without encoding
- Query parameters echoed in page content
- Error messages containing user input

**Stored XSS**

- Database content rendered without sanitization
- User-generated content (comments, profiles, messages) displayed raw
- Rich text fields without allowlist sanitization

**DOM XSS**

- `innerHTML`, `outerHTML`, `document.write()` with dynamic content
- `dangerouslySetInnerHTML` in React without sanitization
- URL fragment (`location.hash`) used in DOM manipulation

Flag: all XSS as HIGH or CRITICAL depending on context.

### Phase 4: Authentication and Authorization

Scan for:

- **Broken authentication**: plaintext password storage, weak hashing
  (MD5, SHA1), missing rate limiting on login, session fixation
- **Broken authorization**: missing access control checks, IDOR
  (direct object reference without ownership validation), privilege
  escalation paths, missing role checks on sensitive operations
- **Session management**: predictable session tokens, missing expiry,
  session not invalidated on logout, cookies without Secure/HttpOnly
- **JWT issues**: none algorithm acceptance, secret in source code,
  missing expiry validation, symmetric signing with weak secret

Flag: all authentication bypass as CRITICAL.

### Phase 5: Insecure Deserialization

Scan for:

- `pickle.loads()`, `yaml.load()` (without SafeLoader), `marshal.loads()`
- Java `ObjectInputStream` without type filtering
- `JSON.parse()` followed by prototype access without validation
- Custom deserializers that instantiate arbitrary classes
- XML external entity (XXE) in XML parsers without entity disabled

Flag: deserialization of untrusted input as CRITICAL.

### Phase 6: Path Traversal and File Operations

Scan for:

- User input in file paths without sanitization (`../` sequences)
- `os.path.join()` with absolute path override
- Missing path canonicalization before access control check
- File upload without extension/type validation
- Temporary file creation with predictable names

Flag: path traversal as HIGH, unrestricted file upload as CRITICAL.

### Phase 7: Server-Side Request Forgery (SSRF)

Scan for:

- User-supplied URLs in server-side HTTP requests
- URL parsing bypass (IP address forms, DNS rebinding)
- Internal service access via user-controlled URLs
- Redirect following without destination validation
- Cloud metadata endpoint access (169.254.169.254)

Flag: SSRF reaching internal services as CRITICAL.

### Phase 8: Security Misconfiguration

Scan for:

- Debug mode enabled in production configurations
- CORS with wildcard origin (`Access-Control-Allow-Origin: *`)
- Missing security headers (CSP, X-Frame-Options, HSTS)
- Default credentials in configuration files
- Verbose error messages exposing stack traces to users
- TLS/SSL configuration weaknesses

Flag: debug mode in production as HIGH, missing CORS restrictions as MEDIUM.

---

## Severity Classification

| Severity | Criteria                                                        | Examples                                                                                |
| -------- | --------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| CRITICAL | Exploitable vulnerability with direct data/system compromise    | SQL injection, RCE, auth bypass, SSRF to internal, deserialization of untrusted data    |
| HIGH     | Exploitable with additional conditions or limited blast radius  | Stored XSS, path traversal, weak password hashing, missing rate limiting                |
| MEDIUM   | Defense-in-depth gap or requires specific conditions to exploit | Reflected XSS with CSP, missing security headers, verbose errors, CORS misconfiguration |
| LOW      | Best practice deviation with minimal direct security impact     | Missing HttpOnly on non-sensitive cookie, informational header leakage                  |

---

## Output Format

````markdown
## Security Review Report

**Files scanned:** <count>
**Attack surface:** <entry_point_count> entry points, <data_flow_count> data flows
**Findings:** <critical> CRITICAL, <high> HIGH, <medium> MEDIUM, <low> LOW

### CRITICAL

#### [S1] <vulnerability type>

- **File:** `path/to/file.ext:line`
- **Category:** OWASP <category>
- **Exploit scenario:** <how an attacker would exploit this>
- **Impact:** <what an attacker gains>
- **Remediation:**
  ```<lang>
  // fixed code
  ```
````

### HIGH

...

### Summary

<overall security posture assessment, 2-3 sentences>

```

---

## Rules

1. Every finding MUST include an exploit scenario — not just "this is insecure"
2. Every finding MUST include remediation code in the target language
3. Map each finding to an OWASP Top 10 category
4. Do not report theoretical vulnerabilities without a concrete data flow
5. Verify that user input actually reaches the vulnerable sink before reporting
6. Check for existing sanitization/validation before flagging
7. Rate severity based on exploitability and impact, not just pattern matching
8. Flag framework-specific issues (Django ORM vs raw SQL, React vs vanilla DOM)

## Rationalizations

| Rationalization | Reality |
|---|---|
| "It's internal only, no external access" | Internal services get compromised via lateral movement — SSRF, stolen credentials, supply chain attacks all start internal |
| "We'll add auth later" | "Later" is after the breach — unauthenticated endpoints in any environment are exploitable from day one |
| "The framework sanitizes input" | Frameworks sanitize for their default context — template injection, SQL in raw queries, and deserialization bypasses are framework-blind |
| "It's behind a VPN" | VPNs are a network boundary, not an authorization layer — compromised VPN credentials grant full access |
| "No one would think to exploit this" | Security through obscurity is not security — automated scanners and bots don't "think", they enumerate |
| "The risk is theoretical" | All exploits were theoretical until the first breach — if data flows from user input to a dangerous sink, it's exploitable |

## Red Flags

- Reporting vulnerabilities without tracing the data flow from source to sink
- Flagging patterns without checking for existing sanitization/validation
- Rating severity based on pattern matching alone without considering exploitability
- No remediation code provided — findings without fixes are not actionable
- Missing OWASP Top 10 mapping for findings
- Ignoring framework-specific security features (CSRF tokens, parameterized queries, CSP headers)

## Verification

- [ ] Every finding includes: vulnerability description, exploit scenario, affected file:line, OWASP category
- [ ] Every finding includes remediation code in the target language
- [ ] Data flow traced: user input → intermediate processing → vulnerable sink
- [ ] Existing sanitization/validation checked before flagging
- [ ] Severity rated by exploitability AND impact, not just pattern match
- [ ] No theoretical vulnerabilities without concrete data flow evidence

---
> Source: [Mathews-Tom/armory](https://github.com/Mathews-Tom/armory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
