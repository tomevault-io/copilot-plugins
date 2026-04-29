## ralph

> This file provides guidance to Claude Code (claude.ai/code), Cursor (cursor-agent), Codex and any AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Cursor (cursor-agent), Codex and any AI agents when working with code in this repository.

## Overview

Ralph is a framework for organizing AI coding assistant workflows. It provides:
- **Plan-first loop:** Markdown todo lists executed by AI assistants (Cursor, Claude Code, or Codex)
- **Orchestration:** Multi-stage pipelines (research → design → implementation → review) with artifact handoffs
- **Agents:** Prebuilt agent profiles for specialized work (research, architect, implementation, code-review, qa, security)
- **Dashboard:** Optional Node UI for monitoring plan execution and artifact generation

Ralph is installed into projects via `./install.sh`. The installer copies shared scripts to `.ralph/`, runtime-specific runners to `.cursor/ralph`, `.claude/ralph`, `.codex/ralph`, `.opencode/ralph`, and agents/rules/skills to `.cursor/agents`, `.claude/agents`, `.opencode/agents`, etc.

> **Symlink note (this repo only):** `.cursor`, `.claude`, `.codex`, and `.opencode` at the repo root are symlinks to `bundle/.cursor`, `bundle/.claude`, `bundle/.codex`, and `bundle/.opencode`. Editing files under `bundle/.cursor/` (or `bundle/.claude/`, `bundle/.codex/`, `bundle/.opencode/`) automatically updates the symlinked paths — no manual copy or sync is needed. Similarly, `.ralph/` files are hardlinked to `bundle/.ralph/`, so edits there are also immediately reflected.

## Key Commands

### Run tests (Bats shell test suite)

```bash
# Full test suite
bash scripts/setup-test-fixtures.sh
bats tests/bats/*.bats

# Run a specific test file
bats tests/bats/run-plan-invoke.bats

# Set required env var for all runs (suppresses usage risk prompt)
RALPH_USAGE_RISKS_ACKNOWLEDGED=1 bats tests/bats/run-plan-unified.bats
```

The test suite uses Bats (Bash Automated Testing System). Tests live in `tests/bats/` and cover:
- Installation and setup (`install.bats`, `install-lib.bats`)
- Plan execution (`run-plan-*.bats`)
- Orchestration (`orchestration-*.bats`)
- Human interaction (`human-interaction.bats`)
- MCP server (`mcp-server.bats`)
- Agent scaffolding (`new-agent.bats`, `bundle-new-agent-scripts.bats`)

### Run the installer

```bash
# Full install (all runtimes + dashboard)
./install.sh

# Install to a specific target repo
./install.sh /path/to/repo

# Claude and shared scripts only (no Cursor/Codex)
./install.sh --claude --shared

# Dry-run (print what would be copied)
./install.sh -n
```

Submodule, subtree, partial installs, and cleanup: [docs/INSTALL.md](docs/INSTALL.md).

### Run a plan (`run-plan.sh`)

Invoke **`.ralph/run-plan.sh`** with **`--plan`** (required). Pass **`--runtime`** unless **`RALPH_PLAN_RUNTIME`** is set or you rely on the interactive runtime prompt (TTY). Pass **`--workspace <path>`** for an explicit repo root; if omitted, the workspace defaults to the current working directory. The parser in `bundle/.ralph/bash-lib/run-plan-args.sh` rejects unknown arguments and does not accept positional workspace or plan paths. See [README.md](README.md) for typical commands and canonical examples.

### Ralph Dashboard (Node UI)

In this repository, develop and test from **`ralph-dashboard/`** at the repo root:

```bash
cd ralph-dashboard
npm install
npm run build
npm start
```

Use **`PORT=8124 npm start`** instead of **`npm start`** when you need a different port.

After **`install.sh`** copies Ralph into another project, the dashboard lives at **`.ralph/ralph-dashboard/`**. From that project root run **`cd .ralph/ralph-dashboard && npm install`**, then **`npm run build`** and **`npm start`** (use **`PORT=8124 npm start`** to override the port).

The dashboard reads plan state, logs, and artifacts from `.ralph-workspace/` and provides a UI for monitoring orchestration runs.

### Validate orchestration schema

```bash
bash scripts/validate-orchestration-schema.sh <orchestration-file.orch.json>
```

## Recommended optional tools

- `fzf` - Install for arrow-key menus in interactive prompts (brew install fzf / apt install fzf). Set RALPH_SKIP_FZF_HINT=1 to silence the install hint.
- `python3` - Required for CLI session resume functionality (captures session IDs from tool output).

## Architecture

### Bundle structure

```
bundle/
  .ralph/              # Shared across all runtimes
    run-plan.sh       # Unified plan executor (required by all)
    orchestrator.sh   # Multi-stage orchestration runner
    orchestration-wizard.sh
    cleanup-plan.sh
    new-agent.sh
    bash-lib/         # Helpers: plan-todo, run-plan-env, run-plan-invoke-*.sh, install-ops.sh, etc.
    mcp-server.sh     # Bash MCP server
    agent-config-tool.sh
  .cursor/
    ralph/            # Cursor-specific runner thin wrapper
    agents/           # Cursor agent profiles
      research/
      architect/
      implementation/
      code-review/
      qa/
      security/
  .claude/
    ralph/            # Claude Code-specific runner thin wrapper
    agents/           # Claude agent profiles (same 6 agents)
    rules/
      no-emoji.md
    skills/
      repo-context/SKILL.md
  .codex/
    ralph/            # Codex-specific runner thin wrapper
    agents/           # Codex agent profiles (same 6 agents)
  .opencode/
    ralph/            # Opencode-specific runner thin wrapper
    agents/           # Opencode agent profiles (same 6 agents)
    rules/
      no-emoji.md
    skills/
      repo-context/SKILL.md
```

### Agent configuration

Each prebuilt agent (research, architect, implementation, code-review, qa, security) is defined by:
- `<agent-id>/config.json` -- Ralph/orchestrator uses this (fields: name, model, description, rules, skills, allowed_tools, output_artifacts)
- `<agent-id>/<agent-id>.md` -- Claude Code native sessions use this (YAML frontmatter + instructions)

Both must be kept in sync. Agents declare:
- **model:** The model id used when running with that agent
- **rules:** Paths to constraint files (e.g., `.claude/rules/no-emoji.md` prevents emoji in code/logs)
- **skills:** Paths to skill definitions (e.g., `.claude/skills/repo-context/SKILL.md` teaches the agent about the project structure)
- **allowed_tools:** (Claude headless only) Comma-separated tool names or JSON array
- **output_artifacts:** Default deliverable paths (supports `{{ARTIFACT_NS}}`, `{{PLAN_KEY}}`, and `{{STAGE_ID}}` templates). These are only used as a fallback when the orchestration stage does not define its own `artifacts` or `outputArtifacts`. When the stage declares its own required artifacts, the agent config `output_artifacts` are ignored entirely for that run.

Validation schema is in `bundle/.claude/agents/README.md` (applies to all runtimes).

### Cost defaults for Claude

To reduce token consumption and keep subscription users on the auth-safe path, Ralph defaults to `CLAUDE_PLAN_BARE=0` and `CLAUDE_PLAN_MINIMAL=1`. `CLAUDE_PLAN_BARE=1` remains an opt-in mode for API-key workflows, while minimal mode composes `--disable-slash-commands`, `--strict-mcp-config`, `--mcp-config '{"mcpServers":{}}'`, `--setting-sources project,local`, and `--tools ...` so Claude starts with a narrower, safer surface without relying on keychain-backed auth. During reset-command invocations, Ralph omits `--disable-slash-commands` so the reset command can execute. Set `CLAUDE_PLAN_MINIMAL_DISABLE_MCP=0` or pass `--claude-allow-mcp` to keep minimal mode but allow project MCP servers (for example Playwright MCP on a QA stage).

Session rotation caps cache growth: `RALPH_PLAN_SESSION_MAX_TURNS` defaults to `8` for Claude, rotating the CLI session after this many invocations. Set to `0` to disable rotation. Other runtimes are unaffected.

### How plans and orchestration work

1. **Single plan:** User writes a `.md` file with tasks like `- [ ] Do this` and `- [x] Done`. The runner picks the next open task, invokes the CLI assistant (Cursor/Claude/Codex), updates the plan, repeats until done.

2. **Orchestration:** A `.orch.json` file defines stages (research → architect → implementation → code-review → qa → security or custom). Each stage has:
   - `id`: stage identifier
   - `runtime`: "cursor" | "claude" | "codex" | "opencode"
   - `agent`: agent name to load
   - `plan`: path to that stage's plan file
   - `artifacts`: required outputs for this stage (array of `{ "path": "...", "required": true }`). This is the primary artifact declaration and overrides the agent config's `output_artifacts` entirely. Paths support `{{ARTIFACT_NS}}`, `{{PLAN_KEY}}`, and `{{STAGE_ID}}` tokens.
   - `outputArtifacts`: alternative/additional artifact declarations (same format; merged with `artifacts`). Wizard-generated files use `artifacts`; use `outputArtifacts` for documentation or legacy compatibility.
   - `inputArtifacts`: paths to artifacts from earlier stages that this stage should read (not verified, only provided as context). Array of `{ "path": "..." }`.
   - `model`: optional model override for this stage; sets `CURSOR_PLAN_MODEL` / `CLAUDE_PLAN_MODEL` / `CODEX_PLAN_MODEL` for the runner invocation.
   - `sessionResume`: boolean; forwards `--cli-resume` or `--no-cli-resume` to `run-plan.sh`.

   The orchestrator runs stages sequentially, verifying artifacts exist before advancing. If a stage defines no `artifacts` and no `outputArtifacts`, the agent config's `output_artifacts` are used as a fallback.

   `parallelStages` is optional and changes the execution model from sequential to wave-based parallel runs followed by an optional sequential tail. Assumptions:
   - When `parallelStages` is absent, existing sequential orchestration behavior is preserved.
   - When `parallelStages` is present, stages are grouped into waves (arrays of stage IDs). Each wave executes in parallel; waves run sequentially.
   - Partial `parallelStages` coverage is supported: stages not listed in any wave run sequentially (in `stages[].id` declaration order) after all waves complete. This sequential tail preserves the declaration order from the orchestration JSON.
   - Deterministic failure semantics: the orchestrator completes the active parallel wave (waits for all stages in the wave), then fails with per-stage status if any stage in that wave fails. Remaining waves and the sequential tail do not run.
   
   When `parallelStages` is present, each wave is a comma-separated string of stage IDs. All stage IDs in waves must exist in `stages[].id`, and no stage ID may appear more than once across all waves. Stages not listed in `parallelStages` run sequentially after all waves, in `stages[]` declaration order. A stage in a parallel wave may declare `loopControl` to loop back to any earlier stage; the loop cycles at the end of the wave.

#### Handoffs

Handoff declarations live on `artifacts` or `outputArtifacts` entries. Use `kind: "handoff"` together with `to: "<target-stage-id>"` to mark a file that should be handed to a later stage. The `to` value must match a declared stage id. In sequential orchestration the target stage must run after the producer in `stages[]` order. With `parallelStages` (including partial coverage with a sequential tail), the target stage must run after the producer in execution order: waves run in order, and stages within the same wave cannot have handoffs between them (same rank); stages in the sequential tail run after all waves, in `stages[]` declaration order.

Non-handoff artifact entries may still set `kind` to `design`, `review`, `research`, or `notes`. Leave `kind` and `to` off ordinary artifact declarations.

Handoff markdown files should follow `bundle/.ralph/handoff.template.md`:
- `# Handoff: <FROM_STAGE> -> <TO_STAGE>`
- `<!-- HANDOFF_META: START -->` / `<!-- HANDOFF_META: END -->` with `from`, `to`, and `iteration`
- `## Tasks` containing unchecked `- [ ]` items for work that should be injected into the next stage plan
- `## Context` and `## Acceptance` for background and success criteria

Before each stage runs, the orchestrator scans incoming handoffs for that stage, resolves template tokens in the artifact path, extracts unchecked tasks from `## Tasks`, and injects them into the stage plan inside guarded `RALPH_HANDOFF` blocks. Identical blocks are skipped, same-iteration content changes replace the stale block, and missing or malformed handoff files are logged as warnings instead of failing the run. Injection is enabled by default with `RALPH_HANDOFFS_ENABLED=1`; set it to `0` to disable the behavior.

3. **Session resume:** With `--cli-resume` or `RALPH_PLAN_CLI_RESUME=1`, the runner stores a runtime-specific `session-id.<runtime>.txt` (and related human-interaction files) under `RALPH_PLAN_SESSION_HOME/<RALPH_PLAN_KEY>/` and reuses the same CLI session on future runs (skips context setup, continues where the assistant left off). When `RALPH_PLAN_SESSION_HOME` is unset, the session root is `${RALPH_PLAN_WORKSPACE_ROOT:-<workspace>/.ralph-workspace}/sessions` (see `bundle/.ralph/bash-lib/run-plan-session.sh`), so files resolve under `<workspace>/.ralph-workspace/sessions/<plan-key>` by default. Set `RALPH_PLAN_SESSION_HOME` explicitly to use a different directory.

### Plan format modes

The runner supports two plan file formats:

1. **Default (markdown checkbox):** Standard Ralph format with lines like `- [ ] Task` and `- [x] Done`. Detected when the file does not start with `---` or lacks a `todos:` array in frontmatter.

2. **Cursor frontmatter:** YAML frontmatter format with a `todos` array:
   ```yaml
   ---
   todos:
     - id: "1"
       content: "Task description"
       status: pending
     - id: "2"
       content: "Another task"
       status: completed
   ---
   # Plan content follows
   ```
   Valid status values: `pending`, `in_progress`, `completed`. The runner updates the frontmatter when completing tasks.

**Format selection:**
- **Auto-detect (default):** The runner examines the file for `---` frontmatter markers and a `todos:` array.
- **Override:** Set `RALPH_PLAN_FORMAT=cursor` or `RALPH_PLAN_FORMAT=default` to force a specific format regardless of content.

**Implementation:** Format detection and manipulation are handled in `bundle/.ralph/bash-lib/plan-todo.sh` using Python 3 with PyYAML when available.

### Key environment variables

- `RALPH_USAGE_RISKS_ACKNOWLEDGED=1` -- Skip the one-time usage risk prompt (set in CI)
- `RALPH_PLAN_WORKSPACE_ROOT` -- Override `.ralph-workspace/` location
- `RALPH_PLAN_CLI_RESUME=1` -- Enable CLI session resume
- `RALPH_ARTIFACT_NS` -- Override artifact namespace (defaults to plan file basename)
- `RALPH_PLAN_KEY` -- Explicit plan namespace (defaults to plan file basename)
- `RALPH_PLAN_SESSION_HOME` -- Directory that holds `session-id.<runtime>.txt`, `pending-human.txt`, and the rest of the session artifacts. When unset, defaults to `${RALPH_PLAN_WORKSPACE_ROOT:-<workspace>/.ralph-workspace}/sessions` (workspace-local). Set explicitly to override (for example `${XDG_STATE_HOME:-${XDG_CONFIG_HOME:-$HOME/.config}}/ralph/sessions` if you prefer the user config area).
- `RALPH_HUMAN_POLL_INTERVAL=2` -- Poll interval (seconds) when waiting for offline human input
- `ORCHESTRATOR_VERBOSE=1` -- Log each orchestrator step to stderr
- `ORCHESTRATOR_DRY_RUN=1` -- Print orchestration steps without running
- `RALPH_PLAN_ALLOW_UNSAFE_RESUME=1` -- Allow CLI resume without an existing session-id.<runtime>.txt; use only in isolated environments to avoid session mix-ups.
- `RALPH_MCP_AUTH_TOKEN` -- When set, the MCP server rejects JSON-RPC tool calls missing the matching `authToken` field (code `-32001`).
- `RALPH_MCP_ALLOWLIST` -- Comma-separated workspace/orchestration path prefixes that the MCP server will accept; requests referencing other locations are rejected.
- `RALPH_PLAN_FORMAT` -- Force plan format: `default` (markdown checkbox) or `cursor` (frontmatter with `todos[]` array). When unset, auto-detects based on file content.

### Human interaction flow

When the runner needs human input:
- **TTY attached:** Prompt interactively on `/dev/tty` and continue
- **Non-TTY (orchestrator, CI):** Write `pending-human.txt` under `${RALPH_PLAN_SESSION_HOME}/${RALPH_PLAN_KEY}/` (when `RALPH_PLAN_SESSION_HOME` is unset, this is under `<workspace>/.ralph-workspace/sessions/<RALPH_PLAN_KEY>` by default), poll until the operator edits `operator-response.txt`, then continue

All Q&A is logged to `human-replies.md` in the session directory for auditing.

### Session storage choices
- **Default location:** When `RALPH_PLAN_SESSION_HOME` is unset, Ralph stores `session-id.<runtime>.txt`, `pending-human.txt`, `operator-response.txt`, and `human-replies.md` under `${RALPH_PLAN_WORKSPACE_ROOT:-<workspace>/.ralph-workspace}/sessions/<plan-key>`. Keeping the default under `.ralph-workspace/sessions/` makes session files reachable from Codex and other sandboxes without relying on home-directory access.
- **Python 3 dependency:** CLI resume relies on the JSON demux helper which is written in Python; if Python 3 is missing the runtime logs `Warning: RALPH_PLAN_CLI_RESUME needs python3 ... running without it.` (see `bundle/.ralph/bash-lib/run-plan-invoke-*.sh`) and continues without resuming the previous session.
- **Override:** Set `RALPH_PLAN_SESSION_HOME` to a directory of your choice (for example `${XDG_STATE_HOME:-${XDG_CONFIG_HOME:-$HOME/.config}}/ralph/sessions`) when you want session files outside the workspace tree. If you use Codex with a custom session home, ensure the sandbox can read that path.
- **Codex-specific note:** `bundle/.codex/ralph/codex-exec-prompt.sh` invokes `codex exec --full-auto` with `--sandbox workspace-write` and, for non-resume runs, `--add-dir` on the workspace `.ralph-workspace/` directory so material under that tree (including `.ralph-workspace/sessions/`) is visible. Resume invocations use `codex exec resume` (with a stored session id, or `--last` when `RALPH_PLAN_ALLOW_UNSAFE_RESUME=1` and bare resume applies) and do not add that extra directory flag; prefer a workspace-visible `RALPH_PLAN_SESSION_HOME` if the resume flow must read session files from inside the Codex process.

### Runtime outputs

Generated plan and orchestration outputs live under `.ralph-workspace/` so runners, dashboards, and automated checks can find them consistently.

- Plan runs write logs under `.ralph-workspace/logs/` and generated artifacts under `.ralph-workspace/artifacts/`.
- Orchestration runs may also write stage-specific files beneath `.ralph-workspace/orchestration-plans/` and read summary data from `.ralph-workspace/logs/`.
- Agent configs that declare `output_artifacts` should treat those paths as fallback deliverables only; orchestration-stage `artifacts` and `outputArtifacts` take precedence when present.

### Interpreting invocation-usage.json

Every plan run appends a record to `.ralph-workspace/logs/<PLAN_KEY>/invocation-usage.json` and writes a final rollup to `plan-usage-summary.json`. The same files are served by the Ralph dashboard under Metrics.

#### Per-runtime token field semantics

| Runtime | `input_tokens` | `output_tokens` | `cache_creation_input_tokens` | `cache_read_input_tokens` |
|---------|---------------|-----------------|-------------------------------|--------------------------|
| claude  | uncached input tokens | output tokens | tokens written to the Anthropic prompt cache | tokens read from the cache |
| cursor  | full input tokens (no cache) | output tokens | 0 | 0 |
| codex   | `input_tokens - cached_input_tokens` from the final `token_count` event | `output_tokens + reasoning_output_tokens` | 0 (Codex does not distinguish creation) | `cached_input_tokens` |
| opencode | sum of `tokens.input` across steps | sum of `tokens.output + tokens.reasoning` | sum of `tokens.cache.write` | sum of `tokens.cache.read` |

#### Additional fields

- `max_turn_total_tokens` -- peak single-turn total token count during the invocation (Codex only, derived from `last_token_usage.total_tokens`; 0 for all other runtimes). High values indicate a context-heavy turn and predict slow or failed runs.
- `cache_hit_ratio` -- `cache_read_input_tokens / (input_tokens + cache_read_input_tokens + cache_creation_input_tokens)`. Values near 1 mean the prompt cache absorbed almost all input cost. Values near 0 on Claude indicate cold starts (no active session) or that session resume is not enabled.

Both fields are shown in the Ralph dashboard as "Cache hit" and "Peak turn" columns in the Plan Metrics and Orchestration Metrics tables, and in the per-plan-folder metric strip on plan cards.

### Dashboard metrics

The Ralph dashboard provides a web UI for monitoring plan execution and token usage:

**Metrics sections:**

1. **Overall metrics:** Aggregated statistics across all plans
   - Total elapsed time, input/output tokens, cache hit ratio
   - Per-runtime breakdowns

2. **Per-plan metrics:** Individual plan statistics
   - Each plan folder shows: elapsed time, tokens, cache hit ratio, peak turn
   - Sortable and filterable by runtime, date, status

3. **Per-orchestration metrics:** Stage-level breakdowns for multi-stage pipelines
   - Shows each stage's contribution to total tokens/time
   - Identifies which stages consumed the most resources

**Access:** The dashboard runs at `http://127.0.0.1:8123` by default (use `PORT=8124 npm start` to override). It reads from `.ralph-workspace/logs/` and `.ralph-workspace/artifacts/`.

**API endpoints:**
- `GET /api/metrics/summary` -- Overall aggregated metrics
- `GET /api/metrics/plan/:planKey` -- Per-plan metrics
- `GET /api/metrics/orchestration/:namespace` -- Per-orchestration stage metrics

## Important patterns

### Adding a new agent

Run `bash .ralph/new-agent.sh` to scaffold a new agent (prompts for name, model, description, rules, skills). This creates:
- `.<runtime>/agents/<agent-id>/config.json`
- `.<runtime>/agents/<agent-id>/<agent-id>.md`
- Rule and skill skeleton directories

Keep config.json and the .md file in sync when editing agent metadata.

### Artifact namespace placeholders

Use these tokens in `artifacts`, `outputArtifacts`, and agent `output_artifacts` paths:

| Token | Env var | Example value | Typical use |
|-------|---------|---------------|-------------|
| `{{ARTIFACT_NS}}` | `RALPH_ARTIFACT_NS` | `code-review` | Namespace from the orchestration JSON or plan basename |
| `{{PLAN_KEY}}` | `RALPH_PLAN_KEY` | `code-review-01-cr1` | Plan namespace from `RALPH_PLAN_KEY` (falls back to `{{ARTIFACT_NS}}` when unset) |
| `{{STAGE_ID}}` | `RALPH_STAGE_ID` | `cr1` | Sanitized stage `id` from the orchestration JSON |

Examples:
- `".ralph-workspace/artifacts/{{ARTIFACT_NS}}/architecture.md"` → `".ralph-workspace/artifacts/my-feature/architecture.md"`
- `".ralph-workspace/artifacts/{{ARTIFACT_NS}}/{{STAGE_ID}}.md"` → `".ralph-workspace/artifacts/code-review/cr1.md"` (when run as stage `cr1`)

### MCP server

The Bash MCP server (`bash .ralph/mcp-server.sh`) exposes plan state and orchestration history as resources, allowing Claude, Cursor, or other MCP clients to query Ralph state without direct file access.

```bash
RALPH_MCP_WORKSPACE="$PWD" bash .ralph/mcp-server.sh
```

Requires `jq` for JSON parsing.

### Validation and error handling

Key scripts to understand:
- `.ralph/bash-lib/install-ops.sh` -- Installer flag parsing and validation
- `.ralph/agent-config-tool.sh` -- Agent config validation and schema checking
- `scripts/validate-orchestration-schema.sh` -- Validates `.orch.json` format

The agent-config-tool is called by all runtimes to verify agents before starting a plan run.

## Testing notes

- **Bats framework:** Each `.bats` file is a standalone test; use `load 'test_helper'` to share helpers
- **Setup/teardown:** Bats provides `setup()` and `teardown()` functions per test
- **Test fixtures:** `scripts/setup-test-fixtures.sh` generates `.ralph-workspace/` stubs for offline testing
- **CI:** GitHub Actions runs `bats tests/bats/*.bats` with `RALPH_USAGE_RISKS_ACKNOWLEDGED=1`

### Running specific tests

```bash
# Run tests matching a pattern
bats tests/bats/run-plan*.bats

# Run one test function
bats tests/bats/orchestration-integration.bats --filter "integration test for multi-stage"
```

## Quick file reference

| File | Purpose |
|------|---------|
| `.ralph/run-plan.sh` | Main plan executor (unified across Cursor/Claude/Codex) |
| `.ralph/orchestrator.sh` | Multi-stage orchestration runner |
| `.ralph/bash-lib/run-plan-invoke-*.sh` | Runtime-specific invoke logic (cursor, claude, codex, opencode) |
| `.ralph/agent-config-tool.sh` | Agent config validation and context building |
| `.ralph/orchestration.template.json` | Starter orchestration plan template |
| `.ralph/plan.template` | Starter plan template |
| `.claude/agents/README.md` | Agent configuration schema documentation |
| `.ralph/ralph-dashboard/` (installed) or `ralph-dashboard/` (this repo) | Dashboard package; `cd` there, then `npm install`, `npm run build`, and `npm start` |
| `scripts/setup-test-fixtures.sh` | Test fixture generator (creates `.ralph-workspace/`) |
| `tests/bats/*.bats` | Bats test files |

## Rules and conventions

- **Agent naming:** Lowercase with hyphens (e.g., `code-review`), no underscores or spaces
- **No emojis:** The `.claude/rules/no-emoji.md` rule (and equivalents in `.cursor`, `.codex`, `.opencode`) forbids emoji in code, comments, and logs
- **Output artifacts:** Every agent should declare at least one output artifact so orchestration can verify completion
- **Session resumption:** Safe only in isolated environments; bare resume (without stored session ID) requires `RALPH_PLAN_ALLOW_UNSAFE_RESUME=1` to prevent session mix-up on shared machines

---
> Source: [JoshJancula/ralph](https://github.com/JoshJancula/ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
