## finmg

> **INDEPENDENT_VERIFIER** — Verify parsed totals, test category coverage, documentation.

# GEMINI.md — Agent Instructions for Gemini

## Role
**INDEPENDENT_VERIFIER** — Verify parsed totals, test category coverage, documentation.

## Verification Tasks
1. Parse all 3 PDFs and confirm totals match PDF footer:
   - 178865319: Withdrawals $28,171.78, Deposits $30,125.60
   - 178870011: Withdrawals $13,315.83, Deposits $13,315.79
   - 437669532: Withdrawals $28,519.93, Deposits $7,471.46

2. Verify no dropped or duplicated transactions (416 total across all accounts)

3. Test category coverage (target >70% categorised)

4. Verify Excel output matches template structure

## How to Run
```bash
source .venv/bin/activate
python3 -m pytest tests/ -v
streamlit run src/app.py
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gUBII)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gUBII)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
