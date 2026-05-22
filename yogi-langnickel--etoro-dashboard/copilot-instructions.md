## etoro-dashboard

> Read this file first before inspecting or changing the project. Then read `docs/agent-memory.md`, which is the compact project memory. Only lazy-load focused files in `docs/memory/` when relevant.

# Etoro Dashboard Agent Instructions

Read this file first before inspecting or changing the project. Then read `docs/agent-memory.md`, which is the compact project memory. Only lazy-load focused files in `docs/memory/` when relevant.

## Project Security Classification

This is a financial dashboard that may read portfolio data and may eventually place orders through the eToro API. Treat the project as security-sensitive even when the GitHub repository is public.

Read `SECURITY.md` before changing authentication, API credentials, eToro integration, persistence, exports, logging, or trading/order behavior.

## Non-Negotiable Rules

- Never commit secrets, API keys, user keys, OAuth tokens, refresh tokens, cookies, private account identifiers, real portfolio exports, brokerage statements, screenshots with private balances, or production `.env` files.
- Never print secrets or full authorization headers in logs, tests, screenshots, status reports, or error messages.
- Keep all eToro credentials server-side. Browser code must not receive API keys, user keys, signing secrets, or privileged tokens.
- Default to read-only behavior. Any trading, order placement, order cancellation, copy-trading, or account mutation must be behind an explicit feature flag, confirmation UI, audit logging, and test/sandbox validation.
- Do not add investment advice, recommendations, autonomous trading, or portfolio management claims unless the user explicitly defines the compliance requirements.
- Use official eToro API documentation as the source of truth for endpoints, authentication, scopes, rate limits, and terms. Verify docs before implementing live API behavior.
- Validate and normalize every external API response before it reaches UI or persistence.
- Redact sensitive data in telemetry and error reporting.
- Do not add dependencies that handle credentials, auth, finance, or cryptography without checking maintenance status and security posture.
- Do not disable type checking, linting, security checks, CSRF protection, auth checks, or TLS verification.
- Do not edit generated, dependency, vendor, or build-output files unless explicitly required.

## Recommended Architecture

- Use a server-side API boundary for all eToro requests.
- Store secrets in local `.env` files, deployment secret stores, or OS keychain tooling, never in source.
- Keep UI components data-only and side-effect-free where possible.
- Separate read-only dashboard features from mutation-capable trading features.
- Add tests for authentication failures, rate limits, malformed API responses, and redaction behavior before adding trading actions.

## Markdown Quality

- Keep Markdown files clean for markdownlint.
- Surround lists with blank lines (`MD032`).
- Use `1.` for each ordered-list item unless the repository lint config explicitly requires sequential numbering (`MD029`).

## Required Checks Before Shipping Financial Features

- Threat model updated in `docs/memory/security.md`.
- API contract notes updated in `docs/memory/etoro-api.md`.
- Validation commands documented in `docs/agent-memory.md`.
- No secret material appears in `git diff`, fixtures, docs, tests, or screenshots.
- Public repo suitability reviewed before pushing.

---
> Source: [Yogi-Langnickel/Etoro-Dashboard](https://github.com/Yogi-Langnickel/Etoro-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
