## ai-repository-security-baseline

> > **Read and follow all rules in [AGENTS.md](../AGENTS.md) — it is the central source of truth for this repository.**

# GitHub Copilot Instructions

> **Read and follow all rules in [AGENTS.md](../AGENTS.md) — it is the central source of truth for this repository.**

## Security Rules

- NEVER suggest code that hardcodes secrets, API keys, tokens, or passwords.
- NEVER suggest reading from or writing to `.env` files (`.env.example` is allowed).
- NEVER generate code that constructs SQL queries via string concatenation — use parameterized queries.
- NEVER suggest `eval()`, `exec()`, or equivalent dynamic code execution with user input.
- NEVER suggest disabling SSL/TLS verification or security headers.
- Always suggest input validation at system boundaries.
- Always suggest output encoding to prevent XSS.
- Flag any pattern that matches common secret formats (API keys, tokens, connection strings).

## Privacy Rules

- Never suggest logging PII (names, emails, IP addresses, phone numbers).
- Suggest data masking or redaction when PII must appear in output.
- Prefer anonymous or pseudonymous identifiers over PII.
- Suggest consent mechanisms when generating data collection code.

## Code Quality

- Follow existing patterns and conventions found in the codebase.
- Suggest typed code (TypeScript over JavaScript, type hints in Python, etc.).
- Suggest error handling for all fallible operations.
- Prefer well-established libraries over obscure ones.
- Suggest tests alongside new functions and methods.

## Accessibility

- Use semantic HTML elements in suggestions.
- Include ARIA attributes where appropriate.
- Ensure interactive elements are keyboard accessible.
- Maintain proper heading hierarchy.
- Suggest adequate color contrast values.

## Testing

- Suggest tests for new functions and methods.
- Include edge cases and error paths in test suggestions.
- Use mocks for external services.
- Follow the Arrange-Act-Assert pattern.

## Dependencies

- Only suggest well-known, widely-used packages.
- Prefer packages with active maintenance and large user bases.
- Check package names carefully — avoid typosquatting.
- Always suggest exact version pinning.

## What NOT to Generate

- Code that accesses files outside the repository.
- Code that makes network requests to hardcoded external URLs.
- Code that disables security features (CSRF, CORS, auth checks).
- Code that uses deprecated or known-vulnerable APIs.
- Infrastructure or CI/CD configuration changes.
- Code containing `TODO` or `FIXME` with security implications.

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
