## matrix-power-nodes-wave1

> This repository is an install bundle for ten independent ComfyUI custom-node packs. The repository

# Zero-context agent contract

This repository is an install bundle for ten independent ComfyUI custom-node packs. The repository
root must not be copied into `custom_nodes`; copy the ten child folders in `custom_nodes/` instead.

## Safe installation procedure

1. Ask for or discover the intended ComfyUI root. Confirm it contains `main.py` and
   `custom_nodes/`.
2. Run `python scripts/verify.py` before any mutation. Stop on any hash or contract failure.
3. Preflight all ten destination names. If one exists, stop and report it. Do not merge, replace,
   rename, move, or delete an existing pack.
4. Use `scripts/install.ps1` on Windows or `scripts/install.sh` on Linux/macOS. The installer copies
   only the ten verified pack folders.
5. Restart ComfyUI, refresh node definitions, and load
   `workflows/matrix-power-nodes-wave1-all-nodes-safe.json` through the visible frontend.
6. Confirm all ten node classes are visible and retain `live=false`.
7. Click a green `enter WaveSpeed Key` row. Confirm a visible `type=password` field appears. Cancel
   it. Never type, read, print, copy, log, screenshot, or inspect a real key without the user's
   explicit request.

## Credential rules

- A key may be entered only through the node's temporary password dialog.
- Never add a key to JSON, widget values, prompts, environment examples, shell history, logs,
  screenshots, issue text, or browser storage.
- Never inspect the local credential file or credential request body.
- Empty Save must report `No API Key entered`; Cancel must make no change.
- A collapsed node must not retain or activate the password field.

## Spend rules

- `live=false` is the only allowed installation and smoke-test state.
- Never enable `live` or make a paid API call unless the user explicitly authorizes that exact run
  and its budget.
- Do not infer paid-call approval from permission to install, verify, or add a key.

## Boundaries

- Do not install dependencies, modify ComfyUI core, change Python/Torch/CUDA, download models, or
  publish to ComfyUI Registry/Manager.
- Do not claim successful installation until exact hashes pass and the real ComfyUI canvas shows
  all ten nodes.
- For support evidence, redact credentials, local user paths, request bodies, private images, and
  provider task identifiers.

---
> Source: [JsonMatrixLab/matrix-power-nodes-wave1](https://github.com/JsonMatrixLab/matrix-power-nodes-wave1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
