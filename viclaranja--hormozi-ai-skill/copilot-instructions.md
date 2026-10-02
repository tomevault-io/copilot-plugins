## hormozi-ai-skill

> python scripts/security_audit.py . --archive release-assets/hormozi-advisor-data-c3380be1.tar.gz

# Project instructions

## Verification

Run:

```bash
python -m pytest -q
python scripts/security_audit.py . --archive release-assets/hormozi-advisor-data-c3380be1.tar.gz
```

## Rules

- The canonical agent skill lives at `skills/ask-hormozi/SKILL.md`.
- Adapters must install byte-identical rendered copies of the canonical skill.
- Business context sources are read-only and local.
- Every attributed answer needs exact YouTube timestamp citations.
- Never add credentials, private business information, private paths, or telemetry.
- Use test-driven development for behavior changes.

---
> Source: [viclaranja/hormozi-ai-skill](https://github.com/viclaranja/hormozi-ai-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
