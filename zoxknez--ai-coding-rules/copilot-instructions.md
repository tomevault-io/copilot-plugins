## 20-security-privacy

> USE WHEN: handling auth, secrets, PII, input validation, or security-sensitive code.


# Security & Privacy Rules

## Secrets & Credentials
- Never hard‑code API keys, passwords, or tokens.
- Use environment variables or secret managers.

## Input Validation
- Validate all external input (Zod/Pydantic or equivalent).
- Reject/normalize unexpected fields.

## Logging
- Never log PII or secrets.
- Scrub sensitive values before logging.

## Safety Stops
- If a change can cause data loss or a breaking change, stop and ask.

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
