## localrl

> Instructions for coding agents that set up or operate LocalRL.

# AGENTS.md

Instructions for coding agents that set up or operate LocalRL.

**One-shot install (preferred entry point):** follow <https://rl.evot.ai/install.md> (same content as [`install.md`](install.md) in this repo).

## Goal

Bring up the **LocalRL web dashboard**. After it is running, all dataset, training, eval, model, and playground work happens in the browser — not via extra CLI training commands.

## Requirements

- Apple Silicon Mac (M-series). Training and inference use MLX.
- Python 3.10+
- Git

## Setup

From a clean machine:

```bash
git clone https://github.com/evotai/localrl
cd localrl
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

If the repo is already cloned, skip clone and use the existing `.venv` when present.

## Start the dashboard

```bash
source .venv/bin/activate
python -m server.app
```

Expected:

```text
LocalRL dashboard -> http://localhost:7600
  (127.0.0.1 only — can launch training jobs; do not expose)
```

Open <http://localhost:7600>.

## What to do in the UI

- **Pipeline**: pick model + dataset pack, run training, watch stages / curves / scoreboard
- **Datasets**: inspect packs and task-signal quality
- **Models**: view / serve released adapters
- **Playground**: compare checkpoints on the same question

Default model: `Qwen3-0.6B-4bit`. Recommended pack for a real run: `datasets/pubmedqa-small`. `pubmedqa-xsmall` is smoke-test only.

## Safety

The writable dashboard can launch local training jobs. Keep it on `127.0.0.1` unless `READ_ONLY=1`.

Public browse-only instance (no launch / cancel / delete / playground runs):

```bash
READ_ONLY=1 LOCALRL_HOST=0.0.0.0 .venv/bin/python -m server.app 7600
```

Online preview: <https://rl.evot.ai>

## Do not

- Do not treat CLI pipeline scripts as the primary UX; the product is the dashboard.
- Do not expose a writable server to untrusted networks.
- Do not use `pubmedqa-xsmall` for release decisions.

---
> Source: [evotai/localrl](https://github.com/evotai/localrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
