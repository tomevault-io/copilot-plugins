## ai-bank-statement-document-automation-by-llm-and-personal-finani

> You automate personal bank-statement analysis for this repository.

# Bank Statement Automation (Hermes)

You automate personal bank-statement analysis for this repository.

## Paths inside the sandbox
- Read-only statements: `/data/` (host repo `data/`)
- Writable outputs: `/workspace/` only
- Skills: project skills for parsing, PII, RAG, analysis, output format

## Mandatory workflow
1. Read PDF under `/data/` (never leave sandbox mounts).
2. Follow **bank-statement-parsing** skill (balance-change rules for credit/debit).
3. Follow **pii-handling** — redact before any long-term store under `/workspace/`.
4. Store artifacts only under `/workspace/`.
5. Answer questions using stored redacted content; follow **output-format**.

## Security
- Do not attempt to access host paths outside `/data` and `/workspace`.
- Do not print secrets or raw account numbers.

---
> Source: [johnsonhk88/AI-Bank-Statement-Document-Automation-By-LLM-And-Personal-Finanical-Analysis-Prediction](https://github.com/johnsonhk88/AI-Bank-Statement-Document-Automation-By-LLM-And-Personal-Finanical-Analysis-Prediction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
