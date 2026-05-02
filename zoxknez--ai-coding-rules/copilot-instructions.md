## security

> > Apply when working with auth, security, or sensitive code paths.

# Windsurf Rules - Security

> Apply when working with auth, security, or sensitive code paths.

## 🔴 Non-Negotiable

- NEVER log tokens, passwords, or API keys
- NEVER commit secrets to version control
- ALWAYS validate and sanitize user input
- ALWAYS use parameterized queries

## Authentication & Authorization

- Verify both authn AND authz
- Use constant-time comparison for secrets
- Implement rate limiting on auth endpoints
- Hash passwords with bcrypt/argon2

## Session Management

- Use httpOnly, secure, sameSite cookies
- Implement proper session invalidation
- Rotate session IDs after login

## API Security

- Validate JWT signatures and expiration
- Set appropriate CORS policies
- Use TLS for all external communication

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
