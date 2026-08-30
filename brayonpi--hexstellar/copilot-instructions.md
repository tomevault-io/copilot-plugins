## hexstellar

> Read AGENTS.md first — it is the canonical instruction file for this repository.

Read AGENTS.md first — it is the canonical instruction file for this repository.

This repository is the PUBLIC thin client for the HexStellar Cortex optimization service. The engine is
proprietary and not present here; never add solver logic, and never describe engine internals in any
public string or comment.

Before completing a change:
- preserve the output contract: stdout is data (JSON when piped), stderr is diagnostics;
- preserve the exit-code contract (0 success · 1 retryable · 2 auth · 3 validation · 130 interrupt);
- an offline solve must ERROR — the client never computes answers locally;
- every doc example must run verbatim (problems require a `description` field);
- run `python3 ../client_contract_check.py` and `python3 ../blackbox_check.py` when working inside the
  monorepo; in the public repo, run the CI workflow's offline contract steps.

---
> Source: [brayonpi/hexstellar](https://github.com/brayonpi/hexstellar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
