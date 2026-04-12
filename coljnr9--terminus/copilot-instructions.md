## terminus

> Protocol for the `feedback` CLI - the single source of truth for TODOs, bugs, and feature requests.

# Feedback Tool Instructions

## 1. Trigger Conditions
Activate this tool when the user's query involves:
- Checking, listing, or searching for TODOs, bugs, feature requests, or feedback.
- Submitting new issues or feedback directly from the chat.
- Updating the status of an existing task (e.g., "mark X as done", "resolve bug Y").
- Deleting feedback items.
- **Rule:** Do NOT use code comments (`// TODO`) for task tracking. Use this tool.

## 2. Command Invocation
The command MUST always be run from the workspace root.

- **Working Directory:** Workspace root (`/home/cole/rust/terminus`)
- **Command Prefix:** `./bin/terminus feedback`

## 3. Supported Actions & Syntax

### Read / List
- **Command:** `list`
- **Options:**
  - `--json`: **ALWAYS USE THIS** for agent parsing.
  - `--slot <N>`: Use local slot database instead of remote (default: remote)
  - `--status <status>`: Optional filter by status
  - `--auto-captured`: Show only auto-captured feedback
- **Example:** `./bin/terminus feedback list --json --status submitted`
- **Note:** The list view now displays LLM-generated titles (if available via auto-triage) instead of raw content truncation.

### Show Details
- **Command:** `show <ID> [--slot <N>]`
- **Purpose:** Retrieve full details for a single item, including `context_json` (viewport, selection, page info) and LLM `analysis` (severity, triage info).
- **ID Format:** Simple number (e.g., `42`) defaults to remote, or use `slot1:42` for local slot, or legacy `remote:42`/`local:42`
- **Example:** `./bin/terminus feedback show 42` or `./bin/terminus feedback show 5 --slot 1`

### Start Work
- **Command:** `work <ID> [--slot <N>]`
- **Purpose:** Streamlines the fix workflow.
  1. Checks out a git branch `fix/feedback-<source>-<id>`.
- **Example:** `./bin/terminus feedback work 42` or `./bin/terminus feedback work 5 --slot 1`

### Create / Submit
- **Command:** `submit <CONTENT> [TYPE] [--slot <N>]`
- **Arguments:**
  - `<CONTENT>`: Quote string describing the issue.
  - `<TYPE>`: `bug`, `feature`, `general`, `praise` (Default: general).
  - `--slot <N>`: Submit to local slot database instead of remote (default: remote).
- **Example:** `./bin/terminus feedback submit "Fix navigation overlap" bug` or `./bin/terminus feedback submit "New feature" feature --slot 1`

### Update
- **Command:** `update <ID> --status <STATUS> [--estimate <TIME>] [--slot <N>]`
- **Arguments:**
  - `<ID>`: Feedback ID (e.g., `42` defaults to remote, or `slot1:42` for local slot).
  - `<STATUS>`: New status string (`submitted`, `reviewed`, `in_queue`, `resolved`, `wont_fix`).
  - `--slot <N>`: Use local slot database instead of remote.
- **Example:** `./bin/terminus feedback update 42 --status "in_queue"` or `./bin/terminus feedback update 5 --status resolved --slot 1`

### Resolve (Mark Complete)
- **Command:** `resolve <ID> [--slot <N>]`
- **Example:** `./bin/terminus feedback resolve 42` or `./bin/terminus feedback resolve 5 --slot 1`

### Delete
- **Command:** `delete <ID> [--slot <N>]`
- **Safety:** **CRITICAL** - Do not run this without explicit user confirmation in the current session.
- **Example:** `./bin/terminus feedback delete 42`

## 4. Output Parsing
- **JSON Mode (`--json`):**
  - The tool returns a JSON object with `items` array, `errors` array, and `count`.
  - Parse the `items` array to present a summary to the user.
  - Summarize the top 3-5 items if the list is long, unless the user asks for all.
  - Look for `context_json` in the response to understand technical context (browser, viewport, selection) when analyzing bugs.
- **Human Mode (Default):**
  - Returns a text table. Only use this if the user specifically asks to see the "raw output".

## 5. Safety & Constraints
- **ID Format:** Simple numeric IDs (e.g., `42`) default to remote. Use `slot1:42` for local slot access, or legacy `remote:42`/`local:42` format.
- **Context:** If the user refers to "that bug" without an ID, first run `list` to find the correct ID before taking action.
- **Default Source:** All commands default to remote (production) unless `--slot` is specified.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coljnr9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/coljnr9)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
