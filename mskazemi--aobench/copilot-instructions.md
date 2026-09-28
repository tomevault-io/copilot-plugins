## aobench

> The agent-facing guide for this repository is **[`AGENTS.md`](../AGENTS.md)** at the repo

# Copilot instructions — AOBench

The agent-facing guide for this repository is **[`AGENTS.md`](../AGENTS.md)** at the repo
root. Read it first: it holds the architecture map, the exact build/test/lint commands, the
invariants that must not be broken, and the list of things that must not be changed without
a design discussion.

This file exists only so GitHub Copilot finds that guide. It is deliberately thin — do not
duplicate content here, because a second divergent copy is worse than none.

Human contributors: start with [`README.md`](../README.md) and
[`CONTRIBUTING.md`](../CONTRIBUTING.md).

Two rules worth repeating, because they are the ones most often broken:

1. **Verify an issue's premise against `main` before writing code.** Run the command the
   issue says fails. Issues go stale.
2. **No network access in tests, and no non-determinism in scoring.** Both are load-bearing
   for a benchmark whose entire value proposition is reproducibility.

---
> Source: [MSKazemi/aobench](https://github.com/MSKazemi/aobench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
