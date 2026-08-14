## bank-of-america-angular-modernization-demo

> This is a synthetic Bank of America interview environment for an Angular

# AGENTS.md

## Purpose

This is a synthetic Bank of America interview environment for an Angular
modernization conversation. It contains no production code, customer data,
credentials, official logos or confidential design assets. Treat every fixture
as non-production.

## Working rules

- Use the Node version declared by the active branch.
- Keep `SecureTransferConfirmation` backward compatible for both consumers.
- Do not add network calls, secrets, telemetry exporters or production access.
- Do not include account, routing, transfer or customer identifiers in
  analytics payloads.
- Keep Retail Banking and Customer Servicing green before opening a pull
  request.
- Prefer one reviewable compatibility slice over a broad framework rewrite.
- Record build, unit, contract and browser evidence in the pull request.
- Write repository-facing analysis, pull-request text and review comments in
  English.
- Stop at a draft pull request. Never merge or deploy without a human code
  owner.

## Required gates

Run `./scripts/gate.sh all` on the active branch. On `main`, this proves the
Angular 14 rollback state. The Angular 18 feature branch adds the compliance,
browser and downstream gates required by the upgrade. State the exact commands
and results in the pull request.

---
> Source: [mtajada/bank-of-america-angular-modernization-demo](https://github.com/mtajada/bank-of-america-angular-modernization-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
