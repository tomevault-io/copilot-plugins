## nestjs-restful-api-template

> This rule focuses on essential security best practices in NestJS, addressing common vulnerabilities such as SQL injection, XSS, CSRF, authentication flaws, and IDOR. It emphasizes robust input validation and sanitization using DTOs and validation pipes, ensuring that all user inputs and file uploads are properly validated and secured. By following these practices, developers can reduce attack surfaces, strengthen authentication and authorization, and build applications that are resilient against common security threats.

- **Security Best Practices:**
  - **Common Vulnerabilities:**
    - **SQL Injection:** Prevent SQL injection by using parameterized queries or an ORM that automatically escapes user inputs.
    - **Cross-Site Scripting (XSS):** Protect against XSS by sanitizing user inputs and encoding outputs.
    - **Cross-Site Request Forgery (CSRF):** Implement CSRF protection using tokens or other mechanisms.
    - **Authentication and Authorization Flaws:** Secure authentication and authorization by using strong passwords, multi-factor authentication, and role-based access control.
    - **Insecure Direct Object References (IDOR):** Prevent IDOR by validating user access to resources before granting access.
  - **Input Validation:**
    - Validate all user inputs to prevent malicious data from entering the system. Use DTOs and validation pipes to enforce input constraints.
    - Sanitize user inputs to remove or escape potentially harmful characters.
    - Validate file uploads to prevent malicious files from being uploaded.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SOMONSOUM)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SOMONSOUM)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
