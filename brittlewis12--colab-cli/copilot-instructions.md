## colab-cli

> >


# colab CLI — Agent Guide

CLI client for Google Colab runtimes. Manages runtime lifecycle, notebook sync, code execution, file transfer, and secrets via bash commands returning structured JSON.

Runtimes are full Linux VMs with a Python kernel, optionally with GPU or TPU accelerators. Shell commands are available through Python's `!` prefix or `subprocess`, but `exec` talks to the Python kernel — it is not a direct shell. `upload`/`download` transfer any file type.

Two commands require user browser interaction: `auth login` (one-time OAuth) and `ensure --drive` (one-time Drive consent). All other commands are fully non-interactive.

## Operating Model

| Command | Purpose |
|---------|---------|
| `ensure` | Create or reconnect to a runtime (GPU/TPU/CPU) |
| `pull` / `push` | Sync notebook source between local `.py` and remote `.ipynb` |
| `run` | Execute saved notebook cells on the remote kernel |
| `exec` | Run ad-hoc Python on the live kernel (ephemeral, not saved) |
| `status` / `ls` | Check runtime/kernel state, compute balance, list all notebooks |
| `diff` | Cell-level diff between local `.py` and remote `.ipynb` |
| `restart` | Restart kernel (clear Python state, keep runtime + packages) |
| `interrupt` | Cooperatively cancel running execution |
| `kill` | Release the runtime (destructive — clears all runtime state) |
| `upload` / `download` | Transfer files to/from the runtime (max ~384 MiB) |

**`run` vs `exec`:** `run` executes the notebook source (the remote `.ipynb`). Use `run --push` after editing to push and execute in one step. `exec` runs throwaway Python on the same kernel — useful for inspection and debugging, but it mutates kernel state without updating notebook source. If you used `exec` to define variables or imports, `restart` clears the drift.

**`exec` is Python, not shell.** The kernel is Python. Shell commands require the `!` prefix:
```bash
colab exec training "!pip install torch"       # correct — shell via Python
colab exec training "!ls /content/data/"       # correct
colab exec training "pip install torch"         # WRONG — Python SyntaxError
```

## First Run

```bash
colab auth login                  # opens browser — user must complete OAuth
colab auth status                 # verify: tier, compute units, eligible GPUs
colab ensure training --gpu t4    # allocate GPU runtime (blocks until ready)
# write training.py
colab run training --push         # push local .py then execute all cells
```

## Common Workflows

**Iterative development:**
```bash
# edit training.py
colab run training --push         # push + execute
# read results from stdout JSON → fix code → repeat
```

**Debugging a failure:**
```bash
colab run training --push         # cell fails → read traceback from data.results
colab exec training "print(x.shape)"  # inspect kernel state
# fix training.py
colab run training --push         # re-run
```

**File I/O:**
```bash
colab upload training data.csv data.csv       # upload to /content/data.csv
colab run training --push                      # training code reads data.csv
colab download training model.pt ./model.pt   # download result
```

**Runtime reclamation:** Colab reclaims idle runtimes after ~30 minutes. If this happens during `run` or `exec`, the WebSocket closes and the command returns an error. Recovery: `colab ensure <name> ...` to allocate a new runtime, then `colab push <name>` to restore from the local `.py` and cached `.ipynb`.

## Agent Rules

1. **Two commands need a browser.** `auth login` (one-time) and `ensure --drive` (one-time per runtime). Everything else is non-interactive.
2. **stdout is one JSON object when invoked as a subprocess** (non-TTY). Use `--json` to force JSON in all contexts. stderr carries progress, warnings, and consent prompts.
3. **Always `ensure` before other notebook commands.** `push`, `pull`, `run`, `exec` all require a live runtime.
4. **Use `run --push` after editing source.** This is the standard edit-execute cycle. `run` executes the remote notebook, not local `.py`.
5. **Use `exec` only for inspection/debugging.** It runs on the Python kernel — shell commands work via `!` prefix, not direct bash. Do not rely on `exec`-created state for reproducible runs.
6. **One command at a time per notebook.** No file locking — concurrent commands cause races.
7. **Names, not paths.** Always `colab run training`, never `colab run training.py`.
8. **Runtimes cost compute units.** Use `kill` when done. Check `auth status` for remaining balance and burn rate.

## Timeout Semantics

`--timeout` is a **per-cell client wait budget** — how long the CLI waits for each cell before returning control. It does NOT cancel or interrupt remote execution. A 5-cell notebook with `--timeout 300` could take up to 1500 seconds total.

If the timeout expires:
- CLI returns `TIMEOUT` (exit 6) with partial results if any
- Remote kernel may still be running — execution is not interrupted
- Check `colab status <name>` → `kernelState` to see if it's still busy
- Use `colab interrupt <name>` to explicitly cancel (cooperative, sends `KeyboardInterrupt`)
- Use `colab kill <name>` to release the runtime entirely (destructive)

## Output Contract

When stdout is not a TTY (subprocess, pipe — the normal case for agents), every command prints exactly one JSON object followed by a newline to **stdout**. Use `--json` to force JSON output even in a TTY:

```json
{"ok": true, "command": "run", "ts": "2026-03-12T10:00:00.000Z", "data": {...}}
```

- `ok` — boolean, always present.
- `command` — string, always present.
- `ts` — ISO 8601 timestamp, always present.
- `data` — object, **omitted** (not null) when the command has no payload.
- `error` — object with `code`, `message`, optional `hint`. **Omitted** on success.
- `data` and `error` can coexist: e.g., `run` with a Python exception includes partial cell outputs in `data` and the error in `error`.

**stderr** carries progress messages, warnings, and consent prompts. Do not parse stderr as data. Do not discard stderr silently — consent prompts and dirty-state warnings are operationally significant.

### Exit Codes

| Exit | Error code | Meaning |
|------|------------|---------|
| 0 | — | Success |
| 1 | `ERROR`, `DIRTY`, `CONFLICT` | General error |
| 2 | `USAGE` | Invalid arguments |
| 3 | `NOT_FOUND` | Notebook/runtime/file not found |
| 4 | `AUTH` | Not logged in or token expired |
| 5 | `QUOTA_EXCEEDED` | Insufficient compute units |
| 6 | `TIMEOUT` | Client wait timeout exceeded (remote execution may still be running) |
| 7 | `EXEC_ERROR` | Python/kernel execution failed |

Exit code 7 means execution failed inside the Python/kernel context — could be a code bug, OOM, missing package, or environment issue. Check `data.results` or `data.result.error` for the traceback.

## Commands

### `colab auth login`

One-time OAuth login. Opens a browser — requires user interaction.

**Response**: `{ "email": "user@example.com", "tier": "SUBSCRIPTION_TIER_PRO" }`

### `colab auth status`

Show auth state, tier, quota, burn rate, and eligible accelerators.

**Response**:
```json
{ "loggedIn": true, "email": "...", "tier": "SUBSCRIPTION_TIER_PRO", "computeUnits": 85.5, "consumptionRateHourly": 1.19, "tokenExpired": false, "eligibleGpus": ["T4", "L4", "A100", "V100"], "eligibleTpus": ["V5E1", "V6E1"] }
```

- `computeUnits`: remaining paid compute unit balance.
- `consumptionRateHourly`: current burn rate across all active runtimes (units/hour). `0` when no runtimes are running.
- `eligibleGpus`: GPU models valid for `--gpu`. `eligibleTpus`: TPU models valid for `--tpu`.

### `colab auth logout`

Revoke tokens and delete stored credentials.

### `colab ensure <name> --gpu <type> | --tpu <type> | --cpu-only`

Allocate a notebook + runtime. Idempotent for the same name + accelerator spec. Exactly one of `--gpu`, `--tpu`, or `--cpu-only` is required.

```bash
colab ensure training --gpu t4
colab ensure training --gpu a100 --high-mem
colab ensure training --tpu v5e1
colab ensure training --cpu-only
colab ensure training --gpu t4 --drive      # enable Drive persistence (browser consent on first use)
```

`--gpu` and `--tpu` take a model argument. `--cpu-only` is a bare flag (no value). Valid GPU models come from `auth status` `eligibleGpus`, TPU models from `eligibleTpus`. Cross-variant errors give helpful hints: `--tpu t4` → "t4 is a GPU model, not a TPU. Did you mean --gpu t4?"

Blocks until the kernel is accessible. If `ok: true`, the runtime is ready immediately.

**Response**:
```json
{ "name": "training", "accelerator": "t4", "endpoint": "gpu-t4-s-xyz", "status": "created", "kernelId": "...", "driveEnabled": true, "driveConsentUrl": "https://..." }
```

- `status`: `"created"` (new) or `"existing"` (already running, specs match).
- `driveEnabled`/`driveConsentUrl`: only present when `--drive` is used.

**Accelerator mismatch**: If the notebook already exists with a different accelerator, returns error with hint to `kill` first.

**Reclamation**: Colab reclaims idle runtimes after ~30 minutes. `ensure` auto-detects and creates a new one. `push` then restores from local cache. If `.colab/` cache is also missing, push creates a fresh notebook from the local `.py`.

### `colab pull <name>`

Download remote `.ipynb`, convert to local `<name>.py` (percent format).

```bash
colab pull training
colab pull training --force     # overwrite even if local .py has unpushed changes
```

Refuses if local `.py` has unpushed changes (`DIRTY` error, exit 1). "Unpushed" means the SHA-256 hash of `<name>.py` differs from the hash recorded at last `push` or `pull`. Use `--force` to overwrite, or `push` first. Run `diff` to see what differs.

**Response**: `{ "name": "training", "pyPath": "training.py", "cells": 5 }`

### `colab push <name>`

Convert local `<name>.py`, merge with remote `.ipynb`, upload.

```bash
colab push training
colab push training --force       # suppress remote-modified warning (merge still runs)
colab push training --no-drive    # skip Drive sync even if enabled
```

Merge preserves cell IDs, outputs, and execution counts from the remote while taking source from local `.py`. Outputs from previous runs survive source edits. Cell matching is heuristic (content-addressed, 4-pass); recommend rerunning after large reorderings.

Works without prior `pull` — if you create `training.py` from scratch and push, cells get fresh IDs and empty outputs.

**Response**: `{ "name": "training", "cells": 5, "merged": true }`

### `colab run <name>`

Execute notebook cells on the runtime. Reads cells from the **remote** `.ipynb`.

```bash
colab run training                        # all cells
colab run training --push                 # push first, then run
colab run training --cell 0               # by 0-based index
colab run training --cell "data_prep"     # by title from # %% marker
colab run training --timeout 600          # client wait timeout in seconds (default: 300)
colab run training --continue-on-error    # execute all cells even if some fail
colab run training --no-drive             # skip Drive sync
```

**Cell addressing** (`--cell`): tries integer parse, then title/name match, then cell ID.

**Outputs are persisted per-cell**: after each cell executes, the remote `.ipynb` is updated. If the process dies after cell 3/5, outputs for cells 0-2 are saved.

**`--continue-on-error`**: all cells execute regardless of failures. `ok` is `false` and exit code is `7` if **any** cell failed. `data.results` contains all results including failures.

**Response**:
```json
{
  "name": "training",
  "cellsExecuted": 3,
  "cellsTotal": 5,
  "results": [
    {
      "index": 0,
      "result": {
        "status": "ok",
        "executionCount": 1,
        "stdout": "hello world\n",
        "stderr": "",
        "outputs": []
      }
    }
  ]
}
```

`outputs` contains rich display objects (mime bundles with `data` dict keyed by MIME type). `error` contains `ename`, `evalue`, `traceback` array when present.

### `colab exec <name> "<code>"`

Execute ad-hoc code on the live kernel. Not saved to the notebook. The default kernel is Python, but shell commands work via `!` prefix or `subprocess`.

```bash
colab exec training "print(x)"
colab exec training "import torch; print(torch.cuda.get_device_name())"
colab exec training "import subprocess; subprocess.run(['nvcc', '--version'])"
colab exec training "result" --timeout 60
```

Code is a single shell argument. For simple one-liners, semicolons work: `"x = 1; print(x)"`. For compound statements (if/for/def), use shell-quoted multiline strings. For anything non-trivial, prefer editing `<name>.py` and using `run --push` instead.

Kernel state persists: variables from `run` are accessible via `exec`, and vice versa. Only `restart` clears state.

**Response**:
```json
{
  "name": "training",
  "result": {
    "status": "ok",
    "executionCount": 5,
    "stdout": "Tesla T4\n",
    "stderr": "",
    "outputs": []
  }
}
```

### `colab kill <name>`

Release the runtime. Preserves local `.py` and `.colab/` cache.

**Response**: `{ "name": "training", "unassigned": true, "stateDeleted": true }`

After `kill`, `ensure` + `push` restores from cache.

### `colab restart <name>`

Restart the kernel. Clears Python state (variables, imports). Preserves runtime (GPU, pip-installed packages in the runtime environment).

**Response**: `{ "name": "training", "kernelId": "..." }`

Note: pip-installed packages survive `restart` but NOT `kill`/reclamation — those allocate a fresh VM.

### `colab interrupt <name>`

Request cooperative cancellation of running execution. Sends `KeyboardInterrupt` to the kernel. Cancellation is best-effort — native code, C extensions, or code that catches `BaseException` may not stop promptly. Check `colab status <name>` to verify the kernel returned to idle.

**Response**: `{ "name": "training", "kernelId": "..." }`

### `colab ls`

List all notebooks and runtime status.

**Response**:
```json
{
  "notebooks": [
    { "name": "training", "accelerator": "t4", "endpoint": "...", "status": "running", "createdAt": "..." }
  ],
  "unmanaged": [
    { "endpoint": "gpu-a100-s-abc", "accelerator": "A100" }
  ]
}
```

`unmanaged` = runtimes in your Colab account not managed by this CLI. Use `adopt` to claim.

### `colab status [<name>]`

Without argument: dashboard with auth state, quota, all notebooks.

With argument: single notebook details.

**Response (dashboard)**:
```json
{
  "auth": { "loggedIn": true, "email": "...", "tier": "...", "tokenExpired": false, "computeUnits": 85.5, "consumptionRateHourly": 1.19 },
  "notebooks": [{ "name": "training", "accelerator": "t4", "status": "running" }],
  "unmanaged": [{ "endpoint": "gpu-a100-s-abc", "accelerator": "A100" }]
}
```

**Response (single notebook)**:
```json
{ "name": "training", "accelerator": "t4", "endpoint": "...", "status": "running", "uptimeSeconds": 3600, "kernelState": "idle", "computeUnits": 85.5, "dirty": true, "createdAt": "...", "driveEnabled": true }
```

- `uptimeSeconds`: seconds since runtime was created. Only present when running.
- `kernelState`: `"idle"`, `"busy"`, or `"unknown"`. Only present when running.
- `computeUnits`: remaining paid compute units (present when auth succeeds).

### `colab diff <name>`

Cell-level diff between local `.py` and remote `.ipynb`.

**Response**:
```json
{
  "name": "training",
  "localCells": 5, "remoteCells": 4,
  "added": 1, "deleted": 0, "modified": 1, "unchanged": 3,
  "cells": [
    { "index": 0, "type": "unchanged", "cellType": "code", "preview": "import torch" },
    { "index": 1, "type": "modified", "cellType": "code", "preview": "lr = 0.001", "remotePreview": "lr = 0.01" }
  ]
}
```

### `colab upload <name> <local> <remote>`

Upload a local file to the runtime. Remote paths are relative to `/content/`. Overwrites existing remote files. Parent directories are created by the Contents API.

```bash
colab upload training data.csv data.csv
colab upload training model.pt models/model.pt
```

**Response**: `{ "name": "training", "local": "data.csv", "remote": "data.csv", "bytes": 1048576 }`

### `colab download <name> <remote> <local>`

Download a file from the runtime. Creates parent directories for the local path. Overwrites existing local files.

```bash
colab download training results.csv ./results.csv
```

**Response**: `{ "name": "training", "remote": "results.csv", "local": "./results.csv", "bytes": 2048 }`

### `colab secrets list`

List available Colab secret key names. Payloads are never exposed in CLI output.

**Response**: `{ "keys": ["HF_TOKEN", "WANDB_API_KEY"] }`

Secrets resolve automatically during execution. Python code calling `google.colab.userdata.get('HF_TOKEN')` is handled transparently. Precedence: environment variable > Colab stored secret > not found error.

### `colab adopt <endpoint> --name <name>`

Bind a name to an existing live runtime. Recovery from `.colab/` deletion, or claiming a runtime created in the Colab web UI.

```bash
colab ls                                      # find the endpoint
colab adopt gpu-t4-s-xyz --name training      # bind it
```

**Response**: `{ "name": "training", "endpoint": "gpu-t4-s-xyz", "accelerator": "t4", "kernelId": "..." }`

Fails with `CONFLICT` if the name is already taken.

## The .py File Format

The local `.py` uses jupytext percent format:

```python
# ---
# jupyter:
#   kernelspec:
#     display_name: Python 3
#     language: python
#     name: python3
# ---

# %%
import torch

# %% [markdown]
# # Training

# %% data_prep
x = torch.randn(100, 10)

# %%
model = torch.nn.Linear(10, 1)
```

- `# %%` = code cell
- `# %% [markdown]` = markdown cell (content is comment-prefixed)
- `# %% title` = named code cell (addressable via `--cell title`)
- IPython magics are auto-commented: `%matplotlib inline` becomes `# %matplotlib inline`
- YAML header is optional — if absent, remote metadata is preserved

Edit with standard file read/write tools. The file is valid Python (magics are commented).

## State Drift Warning

These operations affect different layers independently:

| Operation | Modifies kernel state? | Modifies notebook source? | Modifies local .py? |
|-----------|----------------------|--------------------------|---------------------|
| `exec` | Yes | No | No |
| `run` | Yes | Yes (outputs only) | No |
| `pull` | No | No | Yes |
| `push` | No | Yes (source + merge) | No |
| `restart` | Yes (clears all) | No | No |

If `exec` was used to define variables or import modules, the kernel state diverges from the notebook source. Use `restart` to reset if this causes issues.

## Local Artifacts

`.colab/` directory (created by `ensure`, lives at project root):
- `.colab/notebooks/<name>.json` — runtime state (endpoint, accelerator, hashes)
- `.colab/notebooks/<name>.ipynb` — cached notebook (updated on push/pull)

Safe to delete `.colab/` — recoverable via `ensure` + `push` (from `.py`) or `adopt` (from live runtime). Add `.colab/` to `.gitignore`.

## Error Recovery

Most operational errors include a `hint` field with the recovery command. Always check `error.hint` when present:

| Error code | Exit | Recovery |
|-----------|------|----------|
| `AUTH` | 4 | `colab auth login` |
| `NOT_FOUND` | 3 | `colab ensure <name> --gpu <type>` |
| `QUOTA_EXCEEDED` | 5 | Try `--cpu-only` or a smaller accelerator |
| `EXEC_ERROR` | 7 | Fix code; check `data.result.error.traceback` or `data.results[].result.error` |
| `DIRTY` | 1 | `colab push <name>` or use `--force` |
| `TIMEOUT` | 6 | Client wait budget exceeded — remote execution may still be running. Check `colab status <name>` for `kernelState`, use `colab interrupt <name>` to cancel, or increase `--timeout` |
| `CONFLICT` | 1 | `colab kill <name>` to release the name |

## Verification

After every command:
1. Parse stdout as JSON. If parsing fails, the CLI crashed — report the stderr output.
2. Check `ok` field. If `false`, read `error.code` and `error.hint`.
3. Follow the `hint` — it contains the exact recovery command.
4. On exit code 7, the issue is in the executed code, not the CLI. Read the traceback from `data.result.error` or `data.results[].result.error`.

---
> Source: [brittlewis12/colab-cli](https://github.com/brittlewis12/colab-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
