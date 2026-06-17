## 70-security

> Security rules for the entire repository


Security rules:
- Never hardcode secrets, API keys, tokens, credentials, or private URLs.
- Never log passwords, tokens, raw secrets, or sensitive user content.
- Validate and sanitize all user inputs.
- Treat uploads, URLs, prompts, and external content as untrusted.
- Enforce authentication and authorization checks on protected resources.
- Use least-privilege access patterns.
- Add rate limiting to sensitive or expensive endpoints.
- Use secure defaults.

AI-specific security:
- Add prompt injection resistance where relevant.
- Validate tool inputs and outputs.
- Restrict tool access by policy.
- Protect system instructions and internal configuration.
- Handle unsafe model output before returning it to the user.

Infra/security hygiene:
- Prefer environment variables and secret managers.
- Avoid broad wildcard permissions in cloud configs.
- Do not expose internal services publicly unless required.

Do not:
- Assume client-side validation is enough.
- Return internal error details to end users in production.
- Store plaintext passwords or tokens in databases.
- Use default credentials or weak secrets in any environment.
- Disable HTTPS or certificate validation.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
