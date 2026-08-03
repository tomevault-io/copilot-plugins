## claudeforclaude

> This repository implements **ClaudeForClaude (CLFC)**, a Windows-first session indexer and workspace helper for Claude Code.

# AGENTS.md

## Purpose

This repository implements **ClaudeForClaude (CLFC)**, a Windows-first session indexer and workspace helper for Claude Code.

CLFC treats Claude Code transcript JSONL files as the durable source of session history and builds a lightweight local control plane on top of them.

See `docs/claude-conversation-system.md` for observed transcript structure and implementation implications.

The project is intentionally parallel to CodexForCodex, but Claude Code has different primitives:

- Claude Code uses `CLAUDE.md`, `.claude/`, settings, hooks, agents, skills, and prompt flags.
- Claude Code stores plaintext transcripts under `~/.claude/projects/...`.
- Claude Code resumes sessions through the `claude` CLI, not a Codex app-server.
- Claude Code can append or replace system-prompt text per invocation, but its internal default system prompt is not published and should not be reconstructed.

### Local Claude Code backend

This workspace is expected to run Claude Code through Ollama's Anthropic-compatible endpoint when the user's global Claude Code settings are configured that way:

```text
ANTHROPIC_BASE_URL=http://localhost:11434
ANTHROPIC_AUTH_TOKEN=ollama
```

The current preferred cloud-backed Ollama model is:

```text
gpt-oss:120b-cloud
```

The smaller background/default fast model may be:

```text
gpt-oss:20b-cloud
```

Do not assume Anthropic-hosted Claude models are available unless the user asks to switch back or provides Anthropic authentication. When debugging Claude Code execution, first check whether requests are being routed to Ollama through `ANTHROPIC_BASE_URL`.

---

## Core model

### Claude Code transcript

A Claude Code session recorded under:

```text
%USERPROFILE%\.claude\projects\<project>\<session>.jsonl
```

If `CLAUDE_CONFIG_DIR` is set, it replaces `%USERPROFILE%\.claude` as the Claude config root.

Each transcript may contain top-level metadata such as:

- `sessionId`
- `cwd`
- `timestamp`
- `version`
- message records
- tool call and tool result records

Transcripts are plaintext. Anything Claude Code reads, receives, or prints through tools can land in the transcript. CLFC must never copy full transcript content into its index.

### CLFC session

A CLFC-managed wrapper around a Claude Code session.

Stored metadata includes:

- `session_id`
- `session_name`
- `claude_session_id`
- `cwd`
- `workspace_hash`
- `project_key`
- `created_at`
- optional `display_name`
- optional `preview`
- optional `updated_at`
- optional `transcript_path`
- optional `source_session_id`
- optional `source`
- optional `claude_version`
- optional `model`
- optional `effort`
- optional `permission_mode`
- `session_kind` (`normal | template | worker`)
- optional `template_session_id`
- optional `worker_state` (`idle | leased | done | failed`)

The `preview` field, when present, must be short and user-message-only. Do not index tool outputs, file contents, command output, pasted secrets, or large message bodies.

### Workspace

The current working directory where CLFC is initialized.

Each workspace has:

- local metadata under `.clfc/`
- its own `workspace_hash`
- its own active session pointer
- its own transcript-list cache
- per-session runtime workspaces under `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/`

CLFC distinguishes two directories:

- the **real workspace**, where the user's project files live
- the **session workspace**, where CLFC runs `claude` so it can control session-local `CLAUDE.md`, prompt files, settings overlays, and runtime metadata

When executing Claude Code from the session workspace, CLFC grants access to the real workspace with `--add-dir <real_workspace>`.

---

## Storage layout

### Local workspace metadata

```text
.clfc/
  workspace.json
  cache/
    transcript_list.json
```

### Global CLFC data

Windows default location:

```text
%LOCALAPPDATA%\clfc\
  state.json
  workspaces\
    <workspace_hash>\
      <claude_session_id>.json
```

`state.json` stores global fetch state such as `last_search_time`.

`workspaces/<workspace_hash>/<claude_session_id>.json` stores one indexed Claude Code session record per file.

### Claude Code source data

Windows default location:

```text
%USERPROFILE%\.claude\
  projects\
    <project>\
      <session>.jsonl
```

`clfc fetch` scans this tree directly. If `CLAUDE_CONFIG_DIR` is set, scan `<CLAUDE_CONFIG_DIR>\projects` instead.

### Per-session runtime workspace

```text
%USERPROFILE%\.clfc\
  <workspace_hash>\
    <session_name-or-claude_session_id>\
      session.json
      CLAUDE.md
      system_prompt.md
      settings.json
```

The files above are CLFC-owned copies or overlays. They are not the user's project files.

---

## CLI model

The current implemented slice is intentionally smaller than the long-term command set:

```bash
clfc doctor
clfc init
clfc add <display_name>
clfc add <display_name> <session_id_or_prefix>
clfc add <display_name> --checkout
clfc interactive
clfc interactive --dangerously-skip-permissions
clfc exec "<prompt>"
clfc exec --template <path> --var key=value
clfc exec --fork --checkout-new --display-name <display_name> "<prompt>"
clfc resume <session_id_or_prefix>
clfc resume
clfc resume <session_id_or_prefix> --fork
clfc fork
clfc fork <session_id_or_prefix>
clfc fork <session_id_or_prefix> --checkout-new --display-name <display_name>
clfc checkout <session_id_or_prefix>
clfc current
clfc memory status
clfc memory mode sync
clfc memory mode manual
clfc memory init
clfc memory clone <path>
clfc memory clear
clfc prompt status
clfc prompt render <template> --var key=value
clfc prompt apply <template> --vars-json <json_or_path>
clfc prompt mode off|append|replace
clfc prompt clear
clfc settings show
clfc settings set dangerously-skip-permissions on
clfc settings set permission-mode bypassPermissions
clfc scan
clfc index
clfc list
clfc open
clfc gc
clfc gc --apply
clfc name <session_id_or_prefix> <display_name>
clfc inspect <session_id_or_prefix>
```

These commands are read-only and privacy-preserving. They are the foundation for the broader command set below.

The intended command set is:

```bash
clfc init
clfc fetch
clfc fetch --full
clfc gc
clfc gc --apply
clfc prune
clfc list
clfc list -t
clfc list -a
clfc list --refresh
clfc name <session_id_or_prefix> <display_name>
clfc name <session_id_or_prefix> --clear
clfc status
clfc status -a
clfc status --refresh
clfc info
clfc info <session_name>
clfc info <claude_session_id>
clfc add <session_name>
clfc add <session_name> <claude_session_id>
clfc add <session_name> --idx <n>
clfc clone <source_session_or_claude_session_id> <new_session_name>
clfc delete --idx <n>
clfc checkout <session_name>
clfc checkout <claude_session_id>
clfc checkout <session_id_or_prefix>
clfc current
clfc resume
clfc remove <session_name>
clfc remove <session_name> --force
clfc model --list
clfc interactive
clfc interactive --dangerously-skip-permissions
clfc interactive --permission-mode bypassPermissions
clfc resume <session_id_or_prefix>
clfc resume <session_id_or_prefix> --fork
clfc resume <session_id_or_prefix> --dangerously-skip-permissions
clfc fork
clfc fork <session_id_or_prefix>
clfc settings show
clfc settings set model <model>
clfc settings set effort <level>
clfc settings set permission-mode <mode>
clfc settings set dangerously-skip-permissions on|off
clfc exec [-v|--verbose] "<prompt or command>"
clfc resume [-v|--verbose] <claude_session_id> "<prompt or command>"
clfc memory init
clfc memory clear
clfc memory clone <path>
clfc memory --mode sync
clfc memory --mode manual
clfc prompt init
clfc prompt clear
clfc prompt clone <path>
clfc prompt --mode off
clfc prompt --mode append
clfc prompt --mode replace
clfc template mark <session_name_or_claude_session_id>
clfc template unmark <session_name_or_claude_session_id>
clfc worker spawn <template_session> [--name <worker_name>]
clfc worker list [--template <template_session>]
clfc worker exec [-v|--verbose] <worker_session> "<prompt or command>"
clfc worker reset <worker_session>
clfc worker cleanup <worker_session>
clfc doctor
```

Deprecated or wrong-project commands:

```bash
cfc
```

They should fail clearly and suggest the current CLFC command.

---

## Command behavior

### `clfc init`

Initialize local workspace metadata.

Expected behavior:

- create `.clfc/`
- create `.clfc/workspace.json`
- create `.clfc/cache/`
- do not create `.cfc/`
- do not create or mutate `.claude/`

### `clfc fetch`

Index Claude Code transcript files into the global CLFC store.

Implementation rule:

- read `last_search_time` from `%LOCALAPPDATA%\clfc\state.json`
- resolve the Claude config root from `CLAUDE_CONFIG_DIR`, otherwise `%USERPROFILE%\.claude`
- scan `<claude_config_root>\projects` for session transcript JSONL files
- ignore subagent transcripts, `tool-results`, caches, and non-session JSONL files unless a later implementation explicitly supports them
- parse JSONL records incrementally
- extract `claude_session_id` from `sessionId`, falling back to the transcript filename stem only when needed
- extract `cwd` from transcript records when available
- compute `workspace_hash` from the normalized absolute `cwd`
- compute `project_key` from the path segment under `projects`
- store `transcript_path`, timestamps, Claude version, and a short safe preview when possible
- upsert each record into `workspaces/<workspace_hash>/<claude_session_id>.json`
- update `last_search_time` when the scan completes
- use a small overlap window when re-scanning to avoid missing late writes

### `clfc fetch --full`

Force a full transcript rescan.

Implementation rule:

- ignore the saved `last_search_time`
- scan the entire Claude Code `projects` tree
- still update `last_search_time` after the full scan completes

### `clfc gc`

Find stale indexed records and orphan transcript references.

Implementation rule:

- scan the global CLFC index and Claude Code transcript tree
- classify candidates as:
  - `missing transcript`
  - `stale indexed session`
  - `workspace moved`
  - `orphan transcript`
- keep the default mode as dry-run
- perform CLFC index deletion only when `--apply` is present
- do not delete Claude Code transcript files by default
- treat active broken sessions as warnings and clear `active_session_id` only when an affected CLFC record is actually removed

### `clfc prune`

Alias for `clfc gc`.

### `clfc list`

List indexed sessions for the current workspace.

Implementation rule:

- print the compact one-line session summary by default
- read indexed records from `%LOCALAPPDATA%\clfc\workspaces/<workspace_hash>`
- compare with local `active_session_id`
- do not scan the Claude Code transcript tree by default

### `clfc list --refresh`

Refresh the index, then list indexed sessions.

Implementation rule:

- run the equivalent of `clfc fetch`
- perform normal `clfc list` behavior
- attach refreshed transcript status only when explicitly requested

### `clfc status`

Show the older detailed multi-line session view.

Implementation rule:

- preserve the expanded session layout with workspace header and runtime details
- support `-a` and `--refresh`
- keep `clfc list` as the compact default

### `clfc list -a`

List indexed sessions across all workspaces.

Implementation rule:

- read all workspace directories from the global CLFC store
- do not shorten ids when `-a` is used

### `clfc list -t`

List indexed Claude Code transcripts for the current workspace.

Implementation rule:

- read indexed records for the current `workspace_hash`
- assign **0-based indices**
- write the ordered result to `.clfc/cache/transcript_list.json`
- show full ids when `-a` is combined with `-t`

### `clfc add <session_name>`

Create a new CLFC session record backed by a fresh Claude Code session id.

Implementation rule:

- generate a valid UUID for `claude_session_id`
- store the session as pending until first execution
- on first `clfc exec`, invoke Claude Code with `--session-id <uuid>` and `--name <session_name>`

### `clfc add <session_name> <claude_session_id>`

Create a named CLFC session by tracking an existing Claude Code session id.

### `clfc add <session_name> --idx <n>`

Create a named CLFC session by resolving a Claude Code session id from the latest cached `clfc list -t` result.

### `clfc clone <source_session_or_claude_session_id> <new_session_name>`

Clone an existing session by forking its Claude Code session and copying session-local runtime state.

Implementation rule:

- resolve the source session by CLFC session name first, then by Claude Code session id
- prefer Claude Code's `--fork-session` behavior over copying transcript files directly
- if Claude Code cannot materialize a fork without running a prompt, create a pending clone record with `source_session_id`
- materialize the pending clone on first `clfc exec` by resuming the source with `--fork-session`
- clone the CLFC session workspace into the new session workspace

### `clfc delete --idx <n>`

Delete a cached indexed transcript by transcript-list index.

Implementation rule:

- resolve the Claude Code session id from `.clfc/cache/transcript_list.json`
- remove the indexed record from the global CLFC store
- do not delete the Claude Code transcript unless a future explicit transcript-deletion flag is added

### `clfc checkout <session_name>`

Set the active session for the current workspace.

### `clfc checkout <claude_session_id>`

Set an indexed Claude Code session as the active session target for the current workspace.

Implementation rule:

- resolve the session or indexed transcript from the indexed store using current `workspace_hash`
- store the selected `session_id` in local workspace metadata
- ensure `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/session.json` exists
- initialize runtime metadata fields:
  - `name`
  - `claude_session_id`
  - `session_id`
  - `real_workspace`
  - `memory_mode`
  - `prompt_mode`
  - `model`
  - `effort`
  - `permission_mode`
  - `settings_path`
  - `mcp_config_paths`
  - `additional_options`
- default `memory_mode` to `sync`
- default `prompt_mode` to `off`
- default `permission_mode` to Claude Code's default behavior
- default `additional_options` to an empty list

### `clfc remove <session_name>`

Remove a named CLFC session.

Implementation rule:

- remove the indexed CLFC session record
- keep the Claude Code transcript file by default
- clear `active_session_id` when deleting the active session with `--force`

### `clfc info`

Show information for the currently selected session in this workspace.

### `clfc info <session_name>`

Show information for a named session in the current workspace.

### `clfc info <claude_session_id>`

Show information for a specific indexed Claude Code session.

Implementation rule:

- resolve by session name first
- then resolve by indexed `claude_session_id`
- show transcript path, session workspace path, memory mode, prompt mode, model, effort, and permission mode when available

### `clfc model --list`

Show model-related configuration sources.

Implementation rule:

- inspect relevant Claude Code settings files when present
- show `ANTHROPIC_MODEL` when set
- do not invent a profile system
- do not require Claude Code to be installed just to list local settings

### `clfc interactive`

Launch Claude Code in its native interactive mode.

Implementation rule:

- build and run a `claude` command without `-p`
- default the working directory to the current workspace
- load CLFC launcher defaults from `%LOCALAPPDATA%\clfc\settings.json`
- support per-launch overrides for `--model`, `--effort`, `--permission-mode`, `--dangerously-skip-permissions`, `--allow-dangerously-skip-permissions`, `--resume`, `--continue`, `--name`, `--bare`, and `--add-dir`
- pass arguments after `--` directly to Claude Code
- support `--dry-run` for verification without launching Claude Code

### `clfc resume <session_id_or_prefix>`

Resume an indexed Claude Code session in native interactive mode.

Implementation rule:

- resolve the session from the CLFC index by full id or unique prefix
- default the lookup scope to the current workspace
- support `--all` to resolve across indexed workspaces
- support `--refresh` to rebuild the relevant index before resolving
- launch from the indexed transcript `cwd` when it still exists
- pass `--resume <full_session_id>` to Claude Code
- reuse the same launcher defaults and overrides as `clfc interactive`
- support `--fork` by adding Claude Code's `--fork-session`
- support `--dry-run` for verification without launching Claude Code

### `clfc resume`

Resume the currently checked-out workspace session.

Implementation rule:

- require `.clfc/workspace.json`
- read `active_session_id`
- resolve it against the current workspace index
- use the same launch behavior as `clfc resume <session_id_or_prefix>`

### `clfc fork [session_id_or_prefix]`

Fork an indexed or checked-out Claude Code session in native interactive mode.

Implementation rule:

- default to the checked-out active session when no argument is supplied
- resolve the optional argument using the same rules as `clfc resume`
- pass `--resume <full_session_id> --fork-session` to Claude Code
- reuse the same launcher defaults and overrides as `clfc interactive`
- support `--dry-run` for verification without launching Claude Code

### `clfc checkout <session_id_or_prefix>`

Set the active CLFC session for the current workspace.

Implementation rule:

- resolve the session from the CLFC index by full id, unique prefix, or display name
- create `.clfc/workspace.json` if needed
- store `active_session_id`
- do not mutate Claude Code transcripts
- support `--clear` to remove the active pointer

### `clfc current`

Show the active CLFC session for the current workspace.

Implementation rule:

- read `.clfc/workspace.json`
- resolve `active_session_id` against the index
- show session id, display name, updated time, models, and transcript path

### `clfc memory status`

Show session-local `CLAUDE.md` runtime state.

Implementation rule:

- default to the checked-out active session
- resolve optional session id, unique prefix, or display name when provided
- show runtime workspace path, memory mode, memory path, and whether `CLAUDE.md` exists

### `clfc memory mode sync|manual`

Set session-local memory handling mode.

Implementation rule:

- `sync` means `resume` and `fork` copy the nearest `CLAUDE.md` from the real workspace ancestry into the session runtime workspace before launching
- `manual` means CLFC leaves session-local `CLAUDE.md` untouched before launching
- store the mode in `~/.clfc/<workspace_hash>/<session_id>/session.json`

### `clfc memory init`

Create a session-local `CLAUDE.md` and switch memory mode to `manual`.

### `clfc memory clone <path>`

Copy the given file into session-local `CLAUDE.md` and switch memory mode to `manual`.

### `clfc memory clear`

Remove session-local `CLAUDE.md` and switch memory mode back to `sync`.

### `clfc name <session_id_or_prefix> <display_name>`

Set a CLFC-owned display name for an indexed session.

Implementation rule:

- resolve the session from the CLFC index by full id, unique prefix, or existing display name
- store `display_name` in the indexed session record
- preserve `display_name` across `clfc index` refreshes
- reject duplicate display names in the selected scope
- let `clfc list`, `clfc inspect`, and `clfc resume` resolve sessions by display name
- support `--clear` to remove the display name
- do not mutate Claude Code transcripts

### `clfc settings show`

Show CLFC-owned launcher defaults.

Implementation rule:

- read `%LOCALAPPDATA%\clfc\settings.json`
- do not modify Claude Code's global `~/.claude/settings.json`

### `clfc settings set <key> <value>`

Update a CLFC-owned launcher default.

Supported keys:

- `model`
- `effort`
- `permission-mode`
- `dangerously-skip-permissions`
- `allow-dangerously-skip-permissions`

Implementation rule:

- write to `%LOCALAPPDATA%\clfc\settings.json`
- use `clear`, `none`, `null`, or `unset` to remove string defaults
- use `on` or `off` for boolean defaults
- warn when enabling `dangerously-skip-permissions`
- do not silently edit Claude Code settings

### `clfc memory init`

Create `CLAUDE.md` inside the active session workspace.

Implementation rule:

- require an active session
- create `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/CLAUDE.md`
- switch `memory_mode` to `manual`

### `clfc memory clear`

Remove session-local `CLAUDE.md`.

Implementation rule:

- require an active session
- remove `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/CLAUDE.md`
- switch `memory_mode` back to `sync`

### `clfc memory clone <path>`

Copy the given file into the active session workspace as `CLAUDE.md`.

Implementation rule:

- require an active session
- validate that the source path is a file
- copy it to `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/CLAUDE.md`
- switch `memory_mode` to `manual`

### `clfc memory --mode sync`

Set automatic `CLAUDE.md` sync for the active session.

Implementation rule:

- require an active session
- set `memory_mode` to `sync`
- do not directly edit the session-local `CLAUDE.md` at mode-change time

### `clfc memory --mode manual`

Set manual `CLAUDE.md` handling for the active session.

Implementation rule:

- require an active session
- set `memory_mode` to `manual`
- do not directly edit the session-local `CLAUDE.md` at mode-change time

### `clfc prompt init`

Create `system_prompt.md` inside the active session workspace.

Implementation rule:

- require an active session
- create `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/system_prompt.md`
- default `prompt_mode` to `append`
- do not attempt to reproduce Claude Code's unpublished internal system prompt

### `clfc prompt clear`

Remove session-local `system_prompt.md`.

Implementation rule:

- require an active session
- remove `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/system_prompt.md`
- set `prompt_mode` to `off`

### `clfc prompt clone <path>`

Copy the given file into the active session workspace as `system_prompt.md`.

Implementation rule:

- require an active session
- validate that the source path is a file
- copy it to `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/system_prompt.md`
- default `prompt_mode` to `append`

### `clfc prompt --mode off`

Disable session-local system prompt injection.

### `clfc prompt --mode append`

Append session-local `system_prompt.md` to Claude Code's default prompt.

Implementation rule:

- during execution, pass `--append-system-prompt-file <system_prompt.md>`

### `clfc prompt --mode replace`

Replace Claude Code's default system prompt with session-local `system_prompt.md`.

Implementation rule:

- during execution, pass `--system-prompt-file <system_prompt.md>`
- warn that replacement drops Claude Code's default prompt guidance

### `clfc exec [-v|--verbose] "<prompt or command>"`

Run Claude Code using the active CLFC session.

Implementation rule:

- require an active session
- load session runtime metadata from `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/session.json`
- execute the `claude` process with current working directory set to the session workspace
- always grant access to the real workspace with `--add-dir <real_workspace>`
- before running, search for `CLAUDE.md` from the real workspace upward through parent directories
- when `memory_mode` is `sync` and a source file is found, copy it into the session workspace as `CLAUDE.md`
- when `memory_mode` is `manual`, leave session-local `CLAUDE.md` untouched
- when no `CLAUDE.md` is found in the real workspace ancestry, do nothing
- prepend a short user-message notice that the current cwd is the CLFC session workspace and the real work directory is available through `--add-dir`
- when `prompt_mode` is `append`, pass `--append-system-prompt-file <system_prompt.md>`
- when `prompt_mode` is `replace`, pass `--system-prompt-file <system_prompt.md>`
- when the session is pending, start it with `--session-id <claude_session_id>` and `--name <session_name>`
- otherwise resume it with `--resume <claude_session_id>`
- when `-v` is present, print the underlying `claude` command before execution

Representative command shape:

```bash
claude -p --resume <claude_session_id> --add-dir <real_workspace> [--model <model>] [--effort <effort>] [--permission-mode <mode>] [--append-system-prompt-file <file>] "<prompt>"
```

For a pending new session:

```bash
claude -p --session-id <claude_session_id> --name <session_name> --add-dir <real_workspace> "<prompt>"
```

### `clfc resume [-v|--verbose] <claude_session_id> "<prompt or command>"`

Resume an indexed Claude Code session directly without changing the active CLFC session.

Implementation rule:

- resolve the indexed session in the current workspace
- initialize runtime metadata if it does not exist yet
- follow the same `CLAUDE.md`, system prompt, and `--add-dir` behavior as `clfc exec`
- when `-v` is present, print the underlying `claude` command before execution

### `clfc template mark <session_name_or_claude_session_id>`

Mark an existing session as a reusable template.

Implementation rule:

- set `session_kind` to `template`
- keep the existing Claude Code session and transcript
- clear worker-only metadata on the record

### `clfc template unmark <session_name_or_claude_session_id>`

Remove the template role from a session.

Implementation rule:

- set `session_kind` to `normal`
- clear `template_session_id` and `worker_state`

### `clfc worker spawn <template_session> [--name <worker_name>]`

Create a worker session from a template session.

Implementation rule:

- require the source session to already be marked as a template
- create a pending worker record with a fresh `claude_session_id`
- set:
  - `session_kind = worker`
  - `template_session_id = <template session id>`
  - `worker_state = idle`
- clone the template session workspace into the worker session workspace
- copy runtime settings from the template session to the worker session
- materialize the Claude Code fork on first worker execution with `--resume <template claude_session_id> --fork-session`

### `clfc worker list [--template <template_session>]`

List worker sessions in the current workspace.

Implementation rule:

- list only indexed sessions with `session_kind = worker`
- optionally filter by template
- show current `worker_state`

### `clfc worker exec [-v|--verbose] <worker_session> "<prompt or command>"`

Execute a prompt using a worker session.

Implementation rule:

- require the target session to have `session_kind = worker`
- resolve the source template through `template_session_id`
- refresh the template's `CLAUDE.md` from the real workspace ancestry when the template is in `memory_mode = sync`
- copy `CLAUDE.md`, `system_prompt.md`, `settings.json`, and runtime settings from the template session workspace into the worker session workspace
- materialize a pending worker by resuming the template with `--fork-session`
- set `worker_state = leased` before execution
- set `worker_state = done` on success or `failed` on non-zero exit
- keep existing `clfc exec` semantics unchanged for normal sessions

### `clfc worker cleanup <worker_session>`

Remove a worker session and its CLFC artifacts.

Implementation rule:

- require the target session to have `session_kind = worker`
- delete the worker session workspace under `~/.clfc/<workspace_hash>/...`
- remove the indexed worker record from the global registry
- keep Claude Code transcripts by default
- clear `active_session_id` only if the removed worker was active

### `clfc worker reset <worker_session>`

Reset a worker session back to an idle reusable state.

Implementation rule:

- require the target session to have `session_kind = worker`
- resolve the source template through `template_session_id`
- refresh the template's `CLAUDE.md` from the real workspace ancestry when the template is in `memory_mode = sync`
- copy `CLAUDE.md`, `system_prompt.md`, `settings.json`, and runtime settings from the template session workspace into the worker session workspace
- do not rewrite Claude Code transcript history
- create a fresh pending worker `claude_session_id`
- set `worker_state = idle`

### `clfc doctor`

Report local readiness.

Implementation rule:

- detect whether `claude` is on `PATH`
- show Claude config root
- show CLFC global data root
- show whether current workspace has `.clfc/workspace.json`
- show active session if present
- warn when transcript cleanup settings may remove source transcripts before CLFC users expect it

---

## Session identifiers

CLFC session ids are generated as:

```text
{workspace_hash}\{random_hash}
```

Example:

```text
cff4cc\8f12ab
```

The `workspace_hash` is derived from the normalized absolute real workspace path.

Claude Code session ids should be UUIDs when CLFC creates them. Existing sessions may be accepted as-is if Claude Code can resume them.

---

## Architectural rules

- session discovery must not depend on session names
- transcript scanning is the primary discovery path
- local workspace files are not the source of truth for session existence
- the global CLFC store drives workspace grouping and fetch state
- Claude Code CLI is the execution bridge
- there is no Codex app-server dependency
- transcript list caching remains workspace-local
- Claude Code model selection comes from Claude settings, CLI flags, and environment variables, not from a fake CFC profile system
- `CLAUDE.md` sync is a CLFC convenience layer, not a substitute for Claude Code's normal memory system
- system prompt replacement must be explicit because it drops Claude Code's default prompt guidance
- transcript mutation is not allowed for normal session management
- all path handling must remain Windows-safe

---

## Validation expectations

At minimum, changes should be validated against:

- `clfc init` creating `.clfc/` instead of `.cfc/`
- `clfc fetch` creating or updating:
  - `%LOCALAPPDATA%\clfc\state.json`
  - `%LOCALAPPDATA%\clfc\workspaces\<workspace_hash>\<claude_session_id>.json`
- `clfc fetch --full` scanning all Claude Code transcript files
- `clfc gc` reporting candidates without mutating state by default
- `clfc gc --apply` removing stale CLFC index records without deleting Claude Code transcripts
- `clfc list` reading indexed records without transcript scanning by default in the compact one-line format
- `clfc list --refresh` refreshing from transcripts before listing
- `clfc list -a` showing full ids
- `clfc list -t` assigning 0-based indices and caching the ordered result
- `clfc status` preserving the older detailed multi-line session view
- `clfc info` resolving active session, named session, and Claude Code session id
- `clfc model --list` showing local model configuration sources without inventing Codex profiles
- `clfc interactive --dry-run` building a native Claude Code interactive command
- `clfc interactive --dangerously-skip-permissions --dry-run` adding Claude Code's bypass flag explicitly
- `clfc resume <session-prefix> --dry-run` resolving an indexed session and building `claude --resume <full_session_id>`
- `clfc resume <session-prefix> --fork --dry-run` adding `--fork-session`
- `clfc init` creating local `.clfc/workspace.json`
- `clfc checkout <session-prefix>` storing the active session pointer
- `clfc resume --dry-run` using the checked-out active session
- `clfc fork --dry-run` building `claude --resume <active_session_id> --fork-session`
- `clfc fork <session-prefix> --dry-run` resolving an indexed session and adding `--fork-session`
- `clfc memory status/init/clone/clear/mode` managing session-local `CLAUDE.md` without mutating project `CLAUDE.md`
- `clfc name <session-prefix> <display-name>` storing CLFC-owned display names and preserving them across index refreshes
- `clfc settings set dangerously-skip-permissions on|off` updating CLFC-owned launcher defaults without editing Claude Code settings
- `clfc add` working for:
  - a pending new Claude Code session UUID
  - an explicit Claude Code session id
  - a cached transcript index
- `clfc clone` creating a pending fork or materialized fork without copying transcript files as authoritative history
- `clfc resume` resolving an indexed session and running the session-aware resume flow
- `clfc memory --mode sync|manual` updating `memory_mode` without directly editing `CLAUDE.md`
- `clfc memory init/clone/clear` managing `CLAUDE.md` in the session workspace
- `clfc prompt init/clone/clear` managing `system_prompt.md` in the session workspace
- `clfc prompt --mode off|append|replace` updating session-local prompt injection policy
- `clfc exec` running from the session workspace, syncing `CLAUDE.md` when requested, and granting the real workspace through `--add-dir`
- `clfc template mark/unmark` updating session role metadata without changing existing session behavior
- `clfc worker spawn/list/exec/reset/cleanup` supporting template-based worker sessions without breaking normal sessions
- `clfc delete --idx <n>` removing the indexed CLFC record for a cached transcript
- `clfc remove` removing the indexed CLFC record while preserving Claude Code transcripts by default
- `clfc doctor` reporting readiness without requiring any mutation

---

## Summary

CLFC treats **Claude Code transcripts as the durable session source** and **indexed session records as the local control plane**.

The implementation should optimize for:

- reliable transcript discovery
- fast workspace-local listing
- minimal local metadata under `.clfc/`
- safe handling of plaintext Claude Code application data
- predictable session management through `add`, `checkout`, `remove`, `info`, `fetch`, `exec`, and `resume`

---
> Source: [Longseabear/ClaudeForClaude](https://github.com/Longseabear/ClaudeForClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
