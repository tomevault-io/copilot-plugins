## humanized-social-publisher

> This repository is designed to be configured with an AI coding agent.

# Agent onboarding contract

This repository is designed to be configured with an AI coding agent.

When a user asks you to set up this project:

1. Read `README.md`, `SECURITY.md`, and `config.example.yaml`.
2. Run `./run.sh setup` and let the user choose platforms, account aliases, local content directory, timezone, and schedule slots.
3. Never read, print, copy, commit, upload, or summarize files under `storage/profiles/`.
4. Run `./run.sh doctor` before opening any platform.
5. Run one login command per account: `./run.sh login PLATFORM --account ACCOUNT_KEY`.
6. Keep `safety.require_manual_confirm: true` for the first successful dry run on every account.
7. Use `identity_hint` where the creator page exposes a stable nickname or account ID. Stop on mismatch; never guess the target account.
8. Validate the content package with `./run.sh inspect --post PATH` before queueing it.
9. Do not bypass CAPTCHA, SMS verification, device checks, regional restrictions, rate limits, or platform permissions.
10. Before any Git operation, run `./run.sh sanitize-check`.

Account profiles and `config.yaml` are local state, not project assets.

---
> Source: [lincwang123-bot/humanized-social-publisher](https://github.com/lincwang123-bot/humanized-social-publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
