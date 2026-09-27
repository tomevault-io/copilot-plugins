## glance-speedlab

> This repository is a focused latency lab for live-camera Glance inference. Keep it small, local-first, and measurable.

# Glance Speedlab operating guide

This repository is a focused latency lab for live-camera Glance inference. Keep it small, local-first, and measurable.

## Before changing performance code

1. Add or update an entry in `research/EXPERIMENTS.md` before implementing the experiment.
2. State the hypothesis, primary metric, quality guardrail, hardware, Glance commit/version, and stopping rule.
3. Change one important variable at a time when practical.

Raw run logs live in `research/runs/` and are ignored. Curated summaries and reproducible commands are committed. Never commit camera frames, credentials, model weights, or personally identifying data.

## Checks

```bash
pnpm install
pnpm check
```

Use Node 22 or newer. Run the model separately with prefix sharing enabled:

```bash
glance serve --preload vlm --prefix-cache --port 8077
```

## Boundaries

- `src/web/`: browser capture, scheduler, UI, and client telemetry.
- `src/server/`: thin same-origin gateway and local JSONL telemetry sink.
- `scripts/bench.mjs`: repeatable still-image latency benchmark.
- `research/`: preregistration, evidence, interpretation, and decisions.
- `docs/UPSTREAM.md`: tracks changes that may belong in `glance` or `glance-vlm-demos`.

Prefer native Glance request/response shapes. Do not add an abstraction unless it removes measured overhead or makes experiments more reproducible. Do not call hosted/paid inference paths without explicit user approval.

---
> Source: [yoheinakajima/glance-speedlab](https://github.com/yoheinakajima/glance-speedlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
