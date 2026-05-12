## armorer

> This repository can be operated directly by coding agents such as Codex or Claude Code without going through `armorer chat`.

# Armorer Agent Guide

This repository can be operated directly by coding agents such as Codex or Claude Code without going through `armorer chat`.

## Goal

Use Armorer as the control plane for:

- installing applications
- running and stopping applications
- configuring applications through Armorer-owned flows
- inspecting status, jobs, logs, and config safely

Do not rely on ad hoc shell flows when Armorer already has a first-class command or tool for the task.

## Primary Surfaces

Use these in order of preference:

1. Armorer CLI
2. Armorer-managed config files and registry
3. Direct shell commands only when Armorer does not expose the needed action cleanly

Examples:

```bash
uv run armorer list
uv run armorer install <agent> --yes --skip-scan
uv run armorer run <agent>
uv run armorer uninstall <agent> --force
uv run armorer app configure <agent>
uv run armorer core restart
uv run armorer core clear-queue
uv run armorer core clear-jobs
```

## Where Behavior Lives

Important source files:

- `src/armorer/core.py`
  - chat/core runtime contract
- `src/armorer/tools/orchestrator.py`
  - install/run/uninstall tools
  - background jobs
  - job status and reconciliation
- `src/armorer/installers/dynamic.py`
  - install pipeline
- `src/armorer/registry.py`
  - installed app metadata
- `src/armorer/main.py`
  - CLI entrypoints
- `src/armorer/catalog/playbooks/`
  - app menu/run actions
- `src/armorer/knowledge/`
  - setup and product guidance

## Standard Operating Pattern

For an app lifecycle task, follow this sequence:

1. Inspect current state
2. Decide whether the task is short-running or long-running
3. Use Armorer command/tool
4. Verify the outcome
5. Only then report success

Examples:

- Install:
  1. `uv run armorer list`
  2. if not installed, `uv run armorer install <agent> --yes --skip-scan`
  3. verify with `uv run armorer list` and `uv run armorer run <agent> status` if supported

- Configure:
  1. inspect registry/config files
  2. prefer `uv run armorer app configure <agent>`
  3. verify resulting config and runtime health

- Run:
  1. `uv run armorer run <agent>`
  2. verify containers/processes/status after launch

## Long-Running Operations

Long-running operations should use background job tooling when operating through Armorer Core/chat, and should be treated asynchronously.

Typical long-running tasks:

- install
- uninstall
- long app startup
- long repair flows

Source of truth for background jobs:

- `~/.armorer/jobs/*.json`

Useful fields:

- `status`
- `phase`
- `message`
- `recent_output`
- `heartbeat_at`

Do not trust stale `running` jobs blindly. Reconcile them through Armorer logic rather than assuming they are active.

## Chat and Core

Armorer chat is implemented as:

- client writes tasks to `~/.armorer/inbox`
- Core processes them
- results land in `~/.armorer/outbox`
- session history is stored in `~/.armorer/sessions`

Useful commands:

```bash
uv run armorer chat
uv run armorer core restart
uv run armorer core clear-queue
uv run armorer core clear-jobs
```

If chat behavior looks wrong, inspect:

- `~/.armorer/chat_tui.log`
- `docker logs armorer-core`
- `~/.armorer/jobs/*.json`

## Configuration

Prefer Armorer-owned config flows over app-native interactive setup.

Examples:

- `uv run armorer config`
- `uv run armorer app configure <agent>`

When config is app-specific, inspect the adapter or knowledge files in `src/armorer/apps/` and `src/armorer/knowledge/` before changing behavior.

For channel setup:

- collect the credential first
- then explicitly ask whether Armorer should bind and verify the channel now
- if the user defers binding, apply the config but report channel verification as deferred
- for Telegram-style DM pairing, after binding:
  - tell the user to send a message to the bot
  - ask them to paste the pairing code returned by the bot
  - run the pairing approval command through Armorer/OpenClaw
  - only then report the channel as fully ready

### Setup Q&A Requirement

When the task is to set up or configure an app, do not silently choose user-specific settings unless they are already clearly present.

Use this sequence:

1. inspect the current config and registry state
2. tell the user what is already configured
3. identify only the missing or ambiguous values
4. ask the user a short Q&A, one concrete question at a time
5. apply the config only after the required answers are known
6. verify the resulting runtime state

Typical setup questions include:

- model provider
- exact model id
- API key or base URL
- channels to enable
- skills or tools to enable
- any app-specific credentials required for the chosen setup

If a value is already set, say that explicitly and ask whether to keep it or replace it rather than prompting as if nothing exists.

Do not invent placeholder credentials. Ask for them when needed.

### Required Ordering For Model Setup

When setting up an LLM-backed app, ask in this order:

1. model provider
2. exact model id
3. whether to reuse an existing credential for that provider or replace it
4. if needed, ask for the new API key or base URL

Do not jump from provider selection straight to credential handling.
The model id must be confirmed first.

### Required Handling For Selected Skills

Once the user selects app-native skills, do not stop at writing config.

Use this sequence:

1. inspect the selected skills and their declared requirements
2. filter out skills that are not feasible in the actual runtime context
3. identify missing binaries or credentials
4. install missing binaries in the app's actual runtime context
5. if the app runs in Docker, install or bake the binaries into the runtime container/image context rather than only on the host
6. verify the binaries are available from the runtime context
7. if a selected skill requires auth, login, bootstrap, default account selection, or local app pairing, kick off that process too
8. verify the auth/bootstrap state from the runtime context
9. only then enable the skills in config
10. test that the selected skills are actually usable

Do not treat a host-side install as sufficient for a Dockerized app unless the runtime container can actually use that host binary.

Do not expose skills as selectable if they are not feasible for the current runtime.

Examples of infeasible skills:

- install path only works on the host OS while the app runs in a Linux container
- skill metadata only exposes unsupported installers for the current runtime
- the skill depends on a local desktop app or host integration that the runtime cannot access

For skill verification, prefer checks like:

- runtime `command -v <binary>`
- app status/doctor checks if available
- a minimal safe invocation of the installed CLI from the runtime context

For auth/bootstrap verification, prefer checks like:

- runtime `auth status` / `whoami` / `login status`
- a minimal safe read-only command after auth
- explicit confirmation that required local pairing or account selection completed

## Verification Rules

Never report success from intent alone.

Verify with concrete evidence such as:

- app appears in `armorer list`
- config files updated as expected
- runtime status command succeeds
- container/service is actually running
- health endpoint responds

## Logs and Debugging

Armorer can inspect Docker-backed apps, but prefer structured Armorer flows first.

If needed, use:

```bash
docker ps -a
docker logs <container>
docker compose logs
```

Use shell directly only when Armorer does not expose an equivalent inspection command.

## Data Collection

Training traces are written outside git under:

- `~/.armorer/training_data/raw/`
- `~/.armorer/training_data/exports/sft.jsonl`
- `~/.armorer/training_data/exports/trajectories.jsonl`

Do not delete or overwrite these casually. They are part of the fine-tuning data pipeline.

## Constraints

- Keep operations generic; do not hardcode app-specific logic unless the repository already has a dedicated adapter for that app.
- Prefer reversible/config-aware changes over one-off shell hacks.
- If you add new long-running behavior, make it compatible with Armorer job tracking.
- If you add new chat/runtime behavior, preserve training trace export.

---
> Source: [ArmorerLabs/Armorer](https://github.com/ArmorerLabs/Armorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
