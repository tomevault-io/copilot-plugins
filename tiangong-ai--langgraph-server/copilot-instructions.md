## langgraph-server

> LangGraph server runtime, agent orchestration, gateway, and local development repository.


# ai-langgraph-server AI Working Guide

LangGraph server runtime, agent orchestration, gateway, and local development repository.

This repository is retired from active workspace development as of 2026-09-14.
Read the retirement notice in `README.md` before using historical instructions.
The final source location is `tiangong-ai/langgraph-server`. Workspace detachment
and GitHub archival preserve history; they do not authorize changes to the
shared AI/LCA runtime, cloud resources, data, or credentials. Any future source
reactivation requires a new explicit owner decision and delivery arrangement.

## Local Docpact Push Gate

Install the versioned local hook once per checkout:

```bash
./scripts/install-git-hooks.sh
```

The `pre-push` hook runs `scripts/docpact-gate.sh`, which delegates CLI lookup to `scripts/docpact` and performs strict config validation plus enforced lint before the push leaves the machine. The wrapper checks `DOCPACT_BIN`, Cargo install locations, Homebrew install locations, and then `PATH`, so local agent shells should not fail only because bare `docpact` is unavailable. The default comparison base is `origin/main`. Override it for unusual stacks with `DOCPACT_BASE_REF=<ref>` or `scripts/docpact-gate.sh --base <ref>`. The gate writes its detailed report to a temporary file so normal pushes do not create `.docpact/runs/` artifacts.

---
> Source: [tiangong-ai/langgraph-server](https://github.com/tiangong-ai/langgraph-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
