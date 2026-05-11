## 01-security

> Security rules for preventing secret leakage, injection vulnerabilities, and unsafe code patterns.


# Security Rules

## Secrets & Credentials

- NEVER read, open, or reference .env files (.env.example and .env.template are allowed).
- NEVER hardcode secrets, API keys, tokens, or passwords in any file.
- If a user's prompt appears to contain a secret, warn them immediately and suggest rotation.
- Always use environment variables or secret managers for credentials.

## Secret Patterns to Flag

Warn the user if you detect these patterns in code:

- API keys: `(sk|pk|api|key|token|secret|password|credential)[-_]?[a-zA-Z0-9]{16,}`
- AWS keys: `AKIA[0-9A-Z]{16}`
- Private keys: `-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----`
- Connection strings: `(mongodb|postgres|mysql|redis|amqp)://[^\s]+`
- JWT tokens: `eyJ[A-Za-z0-9-_]+\.eyJ[A-Za-z0-9-_]+`

## Secure Coding

- Validate and sanitize all user input at system boundaries.
- Apply context-appropriate output encoding to prevent XSS.
- Use parameterized queries — never concatenate user input into SQL.
- Never implement custom cryptography — use established libraries.
- Enforce least-privilege access checks on every endpoint.
- Never expose stack traces or internal paths to end users.
- Never use eval(), exec(), or dynamic code execution with user input.
- Never disable SSL/TLS verification or security headers.

## Protected Files (require explicit approval to modify)

- .env, .env.* (except .env.example, .env.template)
- *.pem, *.key, *.cert, *.p12, *.pfx
- .github/workflows/, Dockerfile, docker-compose*.yml
- *.tf, *.tfvars, serverless.yml, k8s manifests
- Lock files (package-lock.json, pnpm-lock.yaml, yarn.lock, etc.)
- CODEOWNERS, .npmrc, .pypirc

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
