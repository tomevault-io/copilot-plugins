## ai-testing-framework

> Enforce ADEF quality gates and CI checks before merge.


# Quality Gates Must Pass

- CI must pass lint, type check, verification script, and gate validation.
- Locally run:
  - python ADEF/scripts/verify_adef_integration.py
  - flake8 ADEF/scripts
  - mypy --config-file mypy.ini ADEF/scripts
  - python ADEF/tools/check_quality_gates.py
- Do not merge if any gate fails.

References:
@.github/workflows/ci.yml
@ADEF/framework/config/quality_gates.yml
@ADEF/tools/check_quality_gates.py
@ADEF/scripts/verify_adef_integration.py

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Manu-rwd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
