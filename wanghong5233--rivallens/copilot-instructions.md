## configuration-management

> Configuration must be centralized, typed, and free of magic numbers across runtime, infra, and Agent flows


# Configuration Management

## What counts as configuration

- Anything tunable per environment: hosts, ports, URLs, credentials, region, feature flags.
- Anything tunable per business: limits, thresholds, timeouts, retry budgets, page sizes, cache TTL, batch sizes.
- Anything tunable per model/provider: model names, endpoints, quotas, fallback chains, prompt versions.
- Anything that you would change without recompiling the product.

## Hard rules

- No magic numbers or magic strings inline. Numeric thresholds, timeouts, sizes, retries, and provider names must be named and centralized.
- Configuration must have a single source of truth per scope (one `Settings` / `config` module per service).
- Configuration values must be typed and validated at process start; the process must fail fast on missing or malformed required values.
- Secrets must come from environment variables or a secret manager. They must not appear in code, tests, fixtures, logs, error messages, or docs.
- Environment-specific behavior must be controlled by configuration, not by `if env == "prod"` branches scattered through code.
- Default values must be safe for local development and explicitly overridable in production.

## Layout

- One typed config object per service or app.
- Group related fields (database, queue, LLM provider, retrieval, storage, auth) into nested sub-objects.
- Provide an `.env.example` (or equivalent) listing every required variable with a short comment.
- Document every config field's unit (ms, s, MB, count) in the field itself, not in prose.

## Anti-patterns

- Reading `os.environ` directly in business logic.
- Hardcoded provider names, model names, or endpoint URLs in service code.
- Different defaults for the same value in different files.
- Boolean flags that silently change critical behavior without being logged at startup.
- "Temporary" inline constants that survive past one PR.

---
> Source: [wanghong5233/RivalLens](https://github.com/wanghong5233/RivalLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
