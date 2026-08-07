## workspace-scope

> Restrict agent work to the LOSPredict project folder only


# LOSPredict workspace scope

This repository is the **only** workspace root. Stay inside it.

## Allowed root

```
C:\Users\1\Documents\LOSPredict
```

All reads, edits, searches, builds, and tests must target paths **under this folder**.

## Do not touch other roots

Unless the user **explicitly** names a path outside LOSPredict, do **not**:

- read, write, delete, or search files outside this repo
- run shell commands whose working directory or targets leave this repo
- modify user-global config (e.g. `~/.cursor`, `~/.gitconfig`, other `Documents\` projects)
- install or change dependencies in parent/sibling directories

If a task seems to require an external path, **stop and ask** instead of assuming.

## Project layout (work here)

| Area | Path | Purpose |
|------|------|---------|
| Python core | `los_predict/` | EKF/UKF/IMM, predictor, simulator, GUI |
| Python demos | `realtime_demo.py`, `run_demo.py` | interactive + batch demos |
| C++ port | `cpp/include/losp/` | header-only library |
| C++ apps | `cpp/apps/` | `run_demo`, `realtime_demo`, `realtime_gui` |
| C++ tests | `cpp/tests/` | unit tests |
| Python tests | `tests/` | pytest suite |

Prefer extending existing modules over adding parallel copies.

## Shell & build

- Default `working_directory`: this repo (or `cpp/` for C++ builds).
- Generated artifacts: `cpp/bin/`, `*.csv`, `*.exe` — do not commit unless asked.
- Do not run destructive git commands (`push --force`, `reset --hard`) unless explicitly requested.

## Commits & docs

- Commit only when the user asks.
- Do not create markdown/docs the user did not request.
- Match existing style: NED frame, radians internally, degrees in CSV/plots.

## When blocked

If the user asks for something outside this folder, reply with what you need and offer to continue **only inside LOSPredict** or after they confirm an external path.

---
> Source: [chiho-chung/LOSPredict](https://github.com/chiho-chung/LOSPredict) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
