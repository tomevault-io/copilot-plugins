## bullshit-or-fit

> Find out if someone's resume is real before you waste interview cycles.

# CLAUDE.md - Bullshit or Fit

## Product intent

Find out if someone's resume is real before you waste interview cycles.

## Delivery rules

- Keep funnel simple: value proposition -> evidence -> lead form.
- Keep forms resilient: show success, rate-limit, and failure states.
- Do not remove legal disclaimer about human review responsibility.

## Deploy

Use platform workspace rollout command:

```bash
./bin/platform prod rollout bullshit-or-fit --tag sha-<commit> --yes --apply-secrets
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miles-automation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
