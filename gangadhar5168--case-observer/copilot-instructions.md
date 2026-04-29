## case-observer

> Analyze user input validation and sanitization mechanisms to identify injection and XSS risks in web applications using PHP/Symfony or Node/Express.


# Rule: Input Validation and Sanitization

## Objective
Analyze how user inputs are validated and sanitized to prevent injection attacks such as XSS, SQLi, and command injection. Recommend fixes focused on strong input validation, safe output encoding, and proper query handling.

## Step 1: Security Analysis and Checklist Creation

1. **Understand Data Flow:** Map where user input enters and how it’s processed/output.
2. **Identify Stack:** PHP with Symfony Validator, or Node/Express with express-validator, etc.
3. **Identify All User Input Points (forms, query params, file names, API payloads).**
4. **Create a Developer-Focused Security Checklist:**

<xml>
### Security Checklist: InputValidator.php / validator.js

#### Code Purpose
Processes user-supplied form data and query parameters.

#### Technology Context
- PHP 8.1 / Symfony Validator
- Node.js with Express and express-validator

#### Attack Surface
- GET/POST form fields, API inputs, file names

#### Security Checklist

- [ ] **Input validation for XSS** ⚠️ CRITICAL
    - Risk: Stored or reflected XSS
    - Complexity: Low
    - Fix: Regex/HTML purifier (PHP) or DOMPurify (Node)

- [ ] **Parameterized queries for SQL** ⚠️ CRITICAL
    - Risk: SQL injection
    - Complexity: Low
    - Fix: PDO (PHP) or mysql2/mongoose param binding

- [ ] **Disallow unsafe characters in file names** ⭐ HIGH ROI
    - Risk: Path traversal or XSS via names
    - Complexity: Low
    - Fix: Strip dangerous chars using regex

- [ ] **Use whitelist validation for types** ⭐ HIGH ROI
    - Risk: Malformed inputs
    - Complexity: Low
    - Fix: Enforce schema with type/range checks

- [ ] **Avoid `eval()` or dynamic exec** ⚠️ CRITICAL
    - Risk: RCE via user input
    - Complexity: High
    - Fix: Refactor logic to avoid `eval`
</xml>

5. **CVE Assessment:**
   - CVE-2023-46734 (Twig unsafe output)
   - CVE-2022-29078 (EJS SSTI)

## Step 2: Security Validation

**Validation Methods:**
1. Code Review
2. XSS payload testing
3. SQLi testing via Burp/sqlmap
4. Input boundary tests

### Example Test Snippet
<xml>
// XSS test payload
<input name="username" value="<img src=x onerror=alert('xss')>">
</xml>

## Step 3: Developer-Focused Remediation

**Example Fix: Express XSS Output Escape**
<xml>
Before:
res.send(`<h1>Welcome, ${req.query.name}</h1>`);

After:
const escape = require('escape-html');
res.send(`<h1>Welcome, ${escape(req.query.name)}</h1>`);
</xml>

**Impact:**
- Prevents script injection
- Ensures safe rendering of user-supplied data

## Final Output Requirements
- Clear validation/encoding checklist
- Before/after code fixes
- Practical tips for devs
- CVE references with fixes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gangadhar5168) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
