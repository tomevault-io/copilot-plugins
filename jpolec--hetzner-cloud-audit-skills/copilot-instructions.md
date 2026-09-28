## hetzner-cloud-audit-skills

> This repository builds a defensive, read-only infrastructure audit tool. Preserve the `collect → reason → verify` boundaries. Never add provider mutation APIs, implicit SSH, live probing, token logging, or shell construction from untrusted metadata.

# Agent guidance

This repository builds a defensive, read-only infrastructure audit tool. Preserve the `collect → reason → verify` boundaries. Never add provider mutation APIs, implicit SSH, live probing, token logging, or shell construction from untrusted metadata.

For every new rule, add evidence requirements, a compensating-control challenge, a positive fixture, and a clean lookalike test. Keep external scanner output as signals. Use synthetic names and documentation IP ranges only.

Before committing: run the standard-library tests, optional lint/type/schema/build checks, demo generation, and Gitleaks. Do not commit `_gtm/`, `.env`, audit outputs from real systems, or credentials.

---
> Source: [jpolec/hetzner-cloud-audit-skills](https://github.com/jpolec/hetzner-cloud-audit-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
