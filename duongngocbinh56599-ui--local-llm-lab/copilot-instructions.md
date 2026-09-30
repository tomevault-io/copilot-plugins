## local-llm-lab

> This repository is designed to stay small, honest, and runnable.

# AGENTS.md

This repository is designed to stay small, honest, and runnable.

## Project intent

local-llm-lab helps developers estimate, deploy dry-runs, benchmark, stress-test, and report on local LLM runs, especially on high unified-memory Apple Silicon machines.

The tool must not overclaim. If a model probably does not fit, say so clearly and give downgrade options.

## Engineering rules

- Keep the core package zero-dependency.
- Optional dependencies must be lazy imports and must never be required for core CLI commands.
- Do not add cloud services, accounts, telemetry, or automatic model downloads.
- Do not collect serial numbers, hardware UUIDs, usernames, credentials, or tokens.
- Label estimates as estimates unless measured by a real backend.
- Prefer deterministic mock/demo data so users can test without downloading giant models.

## Testing

Run:

```bash
python3 -m unittest
```

Useful smoke checks:

```bash
python3 -m local_llm_lab plan --model llama-3.3-70b --quant Q4_K_M --ctx 8192
python3 -m local_llm_lab plan --params 600B --quant Q4_K_M --ctx 32768 --hardware fixture:apple-m4-max-128gb
python3 -m local_llm_lab demo --out examples
```

---
> Source: [duongngocbinh56599-ui/local-llm-lab](https://github.com/duongngocbinh56599-ui/local-llm-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
