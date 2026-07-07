## moto-ocr

> This project processes motorcycle policy images with OCR, matches premium combinations, calculates commissions, and writes settlement outputs.

# AGENTS

## Purpose

This project processes motorcycle policy images with OCR, matches premium combinations, calculates commissions, and writes settlement outputs.

## Settlement Rule Source

- Settlement rules must be maintained in `settlement_rules.json`.
- Do not hardcode frequently changing accident insurance amounts or commission rules in Python.
- `accident_plans` is the most frequently updated section and must stay at the top of `settlement_rules.json`.
- The application logic should treat Python code as the rule engine and `settlement_rules.json` as the business rule source.

## settlement_rules.json Fields

- `accident_plans`: accident insurance plans.
- `accident_plans[].amount`: accident insurance amount.
- `accident_plans[].commission`: commission for that accident insurance amount.
- `compulsory_amounts`: supported compulsory insurance amounts.
- `taxable_compulsory_amounts`: compulsory amounts that may carry tax.
- `monthly_tax`: tax table by month (`1` through `12`).

## Current Business Rules

- `156` compulsory insurance may be combined with monthly tax.
- Tax is not tied to a single total premium amount; it applies by combination.
- Current commission mapping is:
  - `200 -> 0`
  - `302 -> 30`
  - `400 -> 50`
- Premium matching is based on:
  - `total_amount = compulsory + accident + tax`

## Validation Requirements

- Configuration loading must fail fast with clear error messages if `settlement_rules.json` is invalid.
- At minimum, validate:
  - JSON syntax
  - missing required fields
  - duplicate accident plan amounts
  - non-integer or negative amounts where not allowed
  - `taxable_compulsory_amounts` values that are not present in `compulsory_amounts`

## Testing Rules

- Run the full test suite after settlement rule or commission logic changes.
- In the current repo layout, use:

```powershell
$env:PYTHONPATH='.'; pytest -q
```

- Changes to settlement logic are not complete unless tests pass.

## Commit Rules

- Commit messages for settlement changes must be traceable to the business rule change.
- Prefer a subject that states the settlement-rule theme, for example:

```text
Update settlement rules for tax and accident plans
```

- Prefer a body that records the exact rule changes, for example:

```text
- allow tax on all 156 compulsory plans by month
- change accident commission rules: 200->0, 302->30, 400->50
- move settlement rules to JSON config and add validation
```

## Remote Strategy

- Keep `origin` pointing to GitHub for normal development.
- If a Gitee mirror is added, configure it as a separate remote such as `gitee`.
- Prefer explicit pushes like `git push origin master` and `git push gitee master` instead of hiding multiple push targets behind one remote.

---
> Source: [BenjaminNH/moto_ocr](https://github.com/BenjaminNH/moto_ocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
