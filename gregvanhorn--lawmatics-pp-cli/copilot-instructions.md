## lawmatics-pp-cli

> Every Lawmatics resource the API exposes, plus offline full-text search, custom-field reports, and intake analytics... Trigger phrases: `search lawmatics for ...`, `show me overdue tasks in lawmatics`, `bulk reassign lawmatics tasks`, `lawmatics intake report`, `use lawmatics`, `run lawmatics`.


# Lawmatics — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `lawmatics-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install lawmatics --cli-only
   ```
2. Verify: `lawmatics-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails before this CLI has a public-library category, install Node or use the category-specific Go fallback after publish.

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

Lawmatics-pp-cli mirrors your firm's CRM into a local SQLite database so every contact, matter, note, and interaction is searchable in milliseconds. It adds the bulk operations and analytics the Lawmatics UI doesn't: cross-matter overdue task lists, stage bottleneck reports, referral-source revenue, and ad-hoc custom-field pivots straight to CSV.

## When to Use This CLI

Pick this CLI when you need bulk operations across Lawmatics resources, ad-hoc reports the web UI cannot produce, or fast offline search across contacts and matters. Best for intake teams, firm administrators, and developers building automations on top of Lawmatics.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Local state that compounds
- **`search`** — Sub-100ms offline full-text search across contacts, matters, notes, comments, and interactions including custom fields.

  _Reach for this instead of the API when an agent needs cross-entity search by free text._

  ```bash
  lawmatics-pp-cli search "slip and fall" --json
  ```
- **`bottleneck`** — Median matter dwell-time per intake/pipeline stage, surfacing where deals get stuck.

  _Use to answer 'where are matters dying?' without exporting CSVs._

  ```bash
  lawmatics-pp-cli bottleneck --pipeline intake --json
  ```
- **`overdue`** — Every past-due task across every matter, grouped by assignee and sorted by lateness.

  _Standup-ready workload view the Lawmatics UI cannot produce in one click._

  ```bash
  lawmatics-pp-cli overdue --by assignee --json
  ```
- **`since`** — Everything created or updated in the last N hours/days across all entities.

  _Intake-team standup view: what did we touch yesterday?_

  ```bash
  lawmatics-pp-cli since 24h --json
  ```
- **`cf report`** — Join contacts and matters with arbitrary custom fields and pivot to CSV/JSON.

  _Use when the firm needs ad-hoc analytics the web UI's report builder cannot do._

  ```bash
  lawmatics-pp-cli cf report --fields "Case Value,Referral,Stage" --where stage=signed --csv
  ```
- **`velocity`** — Rolling conversion rate (prospect to signed matter) per referral source with sparklines.

  _Marketing ROI question the API answers only with raw lists._

  ```bash
  lawmatics-pp-cli velocity --by source --window 90d --json
  ```
- **`intake stale`** — Prospects with zero interactions in N days.

  _Surfaces leads that are about to die._

  ```bash
  lawmatics-pp-cli intake stale --days 7 --json
  ```
- **`who-touched`** — Unified chronological feed of every interaction, note, task, and email across all matters for a contact.

  _Use before a re-engagement call or status meeting._

  ```bash
  lawmatics-pp-cli who-touched jane.smith@example.com --json
  ```
- **`revenue`** — Sums invoiced and paid amounts per referral source or practice area.

  _Real marketing-channel ROI in one call._

  ```bash
  lawmatics-pp-cli revenue --by source --window ytd --json
  ```
- **`load`** — Open matters, overdue tasks, and billable hours MTD per attorney.

  _Workload balancing in one command._

  ```bash
  lawmatics-pp-cli load --by attorney --json
  ```
- **`explain`** — Markdown brief summarizing a matter from notes, interactions, and tasks.

  _Use before a status call to brief in 5 seconds._

  ```bash
  lawmatics-pp-cli explain MAT-1234 --json
  ```
- **`pipeline drift`** — Matters that skipped a stage or moved backward through the pipeline.

  _Use to find broken intake processes before clients notice._

  ```bash
  lawmatics-pp-cli pipeline drift --json
  ```

### Bulk operations
- **`bulk reassign`** — Reassign every task matching a filter from one user to another in one command.

  _Use during attorney transitions or PTO coverage instead of clicking through hundreds of tasks._

  ```bash
  lawmatics-pp-cli bulk reassign --from alice@firm.com --to bob@firm.com --filter matter.status=open --dry-run
  ```

### Agent-native plumbing
- **`doctor`** — Health check: orphan tasks, contacts with no email, matters in dead stages, custom fields never used, dormant users.

  _Run quarterly to keep the firm's CRM data clean._

  ```bash
  lawmatics-pp-cli doctor --json
  ```
- **`watch`** — Local daemon that polls and POSTs entity diffs to a URL of your choice.

  _Use when integrating Lawmatics with internal automation without paying for native webhooks._

  ```bash
  lawmatics-pp-cli watch contacts --webhook https://hooks.firm.com/lm --since 5m
  ```

## Command Reference

**addresses** — Operations on addresses

- `lawmatics-pp-cli addresses create` — Create an address
- `lawmatics-pp-cli addresses delete` — Delete an address
- `lawmatics-pp-cli addresses get` — Get an address
- `lawmatics-pp-cli addresses list` — List addresses
- `lawmatics-pp-cli addresses update` — Update an address

**comments** — Operations on comments

- `lawmatics-pp-cli comments create` — Create a comment
- `lawmatics-pp-cli comments delete` — Delete a comment
- `lawmatics-pp-cli comments get` — Get a comment
- `lawmatics-pp-cli comments list` — List comments
- `lawmatics-pp-cli comments update` — Update a comment

**companies** — Operations on companies

- `lawmatics-pp-cli companies create` — Create a new company
- `lawmatics-pp-cli companies delete` — Delete a company
- `lawmatics-pp-cli companies find-by-email` — Find a company by email
- `lawmatics-pp-cli companies find-by-name` — Find a company by name
- `lawmatics-pp-cli companies find-by-phone` — Find a company by phone
- `lawmatics-pp-cli companies get` — Get a company by ID
- `lawmatics-pp-cli companies list` — List all companies
- `lawmatics-pp-cli companies update` — Update a company

**contacts** — Operations on contacts

- `lawmatics-pp-cli contacts create` — Create a new contact
- `lawmatics-pp-cli contacts delete` — Delete a contact
- `lawmatics-pp-cli contacts find-by-email` — Find a contact by email
- `lawmatics-pp-cli contacts find-by-name` — Find a contact by name
- `lawmatics-pp-cli contacts find-by-phone` — Find a contact by phone
- `lawmatics-pp-cli contacts get` — Get a contact by ID
- `lawmatics-pp-cli contacts list` — List all contacts
- `lawmatics-pp-cli contacts update` — Update a contact

**custom_contact_types** — Operations on custom contact types

- `lawmatics-pp-cli custom_contact_types create` — Create a custom contact type
- `lawmatics-pp-cli custom_contact_types delete` — Delete a custom contact type
- `lawmatics-pp-cli custom_contact_types get` — Get a custom contact type
- `lawmatics-pp-cli custom_contact_types list` — List custom contact types
- `lawmatics-pp-cli custom_contact_types update` — Update a custom contact type

**custom_fields** — Operations on custom fields

- `lawmatics-pp-cli custom_fields create` — Create a custom field
- `lawmatics-pp-cli custom_fields delete` — Delete a custom field
- `lawmatics-pp-cli custom_fields get` — Get a custom field
- `lawmatics-pp-cli custom_fields list` — List custom fields
- `lawmatics-pp-cli custom_fields update` — Update a custom field

**email_addresses** — Operations on email addresses

- `lawmatics-pp-cli email_addresses create` — Create an email address
- `lawmatics-pp-cli email_addresses delete` — Delete an email address
- `lawmatics-pp-cli email_addresses get` — Get an email address
- `lawmatics-pp-cli email_addresses list` — List email addresses
- `lawmatics-pp-cli email_addresses update` — Update an email address

**email_campaigns** — Operations on email campaigns

- `lawmatics-pp-cli email_campaigns get` — Get an email campaign
- `lawmatics-pp-cli email_campaigns list` — List email campaigns
- `lawmatics-pp-cli email_campaigns stats` — Get email campaign stats

**event_types** — Operations on event types

- `lawmatics-pp-cli event_types create` — Create an event type
- `lawmatics-pp-cli event_types delete` — Delete an event type
- `lawmatics-pp-cli event_types get` — Get an event type
- `lawmatics-pp-cli event_types list` — List event types
- `lawmatics-pp-cli event_types update` — Update an event type

**events** — Operations on events

- `lawmatics-pp-cli events create` — Create an event
- `lawmatics-pp-cli events delete` — Delete an event
- `lawmatics-pp-cli events get` — Get an event
- `lawmatics-pp-cli events list` — List events

**expenses** — Operations on expenses

- `lawmatics-pp-cli expenses create` — Create an expense
- `lawmatics-pp-cli expenses delete` — Delete an expense
- `lawmatics-pp-cli expenses get` — Get an expense
- `lawmatics-pp-cli expenses list` — List expenses
- `lawmatics-pp-cli expenses update` — Update an expense

**files** — Operations on files

- `lawmatics-pp-cli files delete` — Delete a file
- `lawmatics-pp-cli files download` — Download a file
- `lawmatics-pp-cli files get` — Get a file
- `lawmatics-pp-cli files list` — List files
- `lawmatics-pp-cli files update` — Update a file
- `lawmatics-pp-cli files upload` — Upload a file

**folders** — Operations on folders

- `lawmatics-pp-cli folders create` — Create a folder
- `lawmatics-pp-cli folders delete` — Delete a folder
- `lawmatics-pp-cli folders get` — Get a folder
- `lawmatics-pp-cli folders list` — List folders
- `lawmatics-pp-cli folders update` — Update a folder

**interactions** — Operations on interactions

- `lawmatics-pp-cli interactions create` — Create an interaction
- `lawmatics-pp-cli interactions delete` — Delete an interaction
- `lawmatics-pp-cli interactions get` — Get an interaction
- `lawmatics-pp-cli interactions list` — List interactions
- `lawmatics-pp-cli interactions update` — Update an interaction

**invoices** — Operations on invoices

- `lawmatics-pp-cli invoices get` — Get an invoice
- `lawmatics-pp-cli invoices list` — List invoices

**locations** — Operations on locations

- `lawmatics-pp-cli locations` — List locations

**matter_sub_statuses** — Operations on matter sub statuses

- `lawmatics-pp-cli matter_sub_statuses create` — Create a matter sub status
- `lawmatics-pp-cli matter_sub_statuses delete` — Delete a matter sub status
- `lawmatics-pp-cli matter_sub_statuses update` — Update a matter sub status

**matters** — Operations on matters

- `lawmatics-pp-cli matters create` — Create a new matter
- `lawmatics-pp-cli matters delete` — Delete a matter
- `lawmatics-pp-cli matters find-by-email` — Find a matter by email
- `lawmatics-pp-cli matters get` — Get a matter by ID
- `lawmatics-pp-cli matters list` — List all matters
- `lawmatics-pp-cli matters update` — Update a matter

**notes** — Operations on notes

- `lawmatics-pp-cli notes create` — Create a note
- `lawmatics-pp-cli notes delete` — Delete a note
- `lawmatics-pp-cli notes get` — Get a note
- `lawmatics-pp-cli notes list` — List notes
- `lawmatics-pp-cli notes update` — Update a note

**phone_numbers** — Operations on phone numbers

- `lawmatics-pp-cli phone_numbers create` — Create a phone number
- `lawmatics-pp-cli phone_numbers delete` — Delete a phone number
- `lawmatics-pp-cli phone_numbers get` — Get a phone number
- `lawmatics-pp-cli phone_numbers list` — List phone numbers
- `lawmatics-pp-cli phone_numbers update` — Update a phone number

**pipelines** — Operations on pipelines

- `lawmatics-pp-cli pipelines get` — Get a pipeline by ID
- `lawmatics-pp-cli pipelines list` — List pipelines

**practice_areas** — Operations on practice areas

- `lawmatics-pp-cli practice_areas create` — Create a practice area
- `lawmatics-pp-cli practice_areas delete` — Delete a practice area
- `lawmatics-pp-cli practice_areas get` — Get a practice area
- `lawmatics-pp-cli practice_areas list` — List practice areas
- `lawmatics-pp-cli practice_areas update` — Update a practice area

**prospects** — Operations on prospects

- `lawmatics-pp-cli prospects create` — Create a prospect
- `lawmatics-pp-cli prospects delete` — Delete a prospect
- `lawmatics-pp-cli prospects get` — Get a prospect
- `lawmatics-pp-cli prospects list` — List prospects
- `lawmatics-pp-cli prospects update` — Update a prospect

**relationship_types** — Operations on relationship types

- `lawmatics-pp-cli relationship_types create` — Create a relationship type
- `lawmatics-pp-cli relationship_types get` — Get a relationship type
- `lawmatics-pp-cli relationship_types list` — List relationship types
- `lawmatics-pp-cli relationship_types update` — Update a relationship type

**relationships** — Operations on relationships

- `lawmatics-pp-cli relationships create` — Create a relationship
- `lawmatics-pp-cli relationships delete` — Delete a relationship
- `lawmatics-pp-cli relationships get` — Get a relationship
- `lawmatics-pp-cli relationships list` — List relationships
- `lawmatics-pp-cli relationships update` — Update a relationship

**stages** — Operations on stages

- `lawmatics-pp-cli stages get` — Get a stage
- `lawmatics-pp-cli stages list` — List stages

**subtasks** — Operations on subtasks

- `lawmatics-pp-cli subtasks create` — Create a subtask
- `lawmatics-pp-cli subtasks delete` — Delete a subtask
- `lawmatics-pp-cli subtasks get` — Get a subtask
- `lawmatics-pp-cli subtasks list` — List subtasks
- `lawmatics-pp-cli subtasks update` — Update a subtask

**tags** — Operations on tags

- `lawmatics-pp-cli tags create` — Create a tag
- `lawmatics-pp-cli tags delete` — Delete a tag
- `lawmatics-pp-cli tags get` — Get a tag
- `lawmatics-pp-cli tags list` — List tags
- `lawmatics-pp-cli tags update` — Update a tag

**task_statuses** — Operations on task statuses

- `lawmatics-pp-cli task_statuses create` — Create a task status
- `lawmatics-pp-cli task_statuses delete` — Delete a task status
- `lawmatics-pp-cli task_statuses get` — Get a task status
- `lawmatics-pp-cli task_statuses list` — List task statuses
- `lawmatics-pp-cli task_statuses update` — Update a task status

**tasks** — Operations on tasks

- `lawmatics-pp-cli tasks create` — Create a task
- `lawmatics-pp-cli tasks delete` — Delete a task
- `lawmatics-pp-cli tasks get` — Get a task
- `lawmatics-pp-cli tasks list` — List tasks
- `lawmatics-pp-cli tasks update` — Update a task

**time_entries** — Operations on time entries

- `lawmatics-pp-cli time_entries create` — Create a time entry
- `lawmatics-pp-cli time_entries delete` — Delete a time entry
- `lawmatics-pp-cli time_entries get` — Get a time entry
- `lawmatics-pp-cli time_entries list` — List time entries
- `lawmatics-pp-cli time_entries update` — Update a time entry

**transactions** — Operations on transactions

- `lawmatics-pp-cli transactions create` — Create a transaction
- `lawmatics-pp-cli transactions get` — Get a transaction
- `lawmatics-pp-cli transactions list` — List transactions

**users** — Operations on users

- `lawmatics-pp-cli users create` — Create a user
- `lawmatics-pp-cli users delete` — Delete a user
- `lawmatics-pp-cli users get` — Get a user
- `lawmatics-pp-cli users list` — List users
- `lawmatics-pp-cli users update` — Update a user


### Finding the right command

When you know what you want to do but not which command does it, ask the CLI directly:

```bash
lawmatics-pp-cli which "<capability in your own words>"
```

`which` resolves a natural-language capability query to the best matching command from this CLI's curated feature index. Exit code `0` means at least one match; exit code `2` means no confident match — fall back to `--help` or use a narrower query.

## Recipes


### Find stalled prospects

```bash
lawmatics-pp-cli intake stale --days 14 --json --select id,name,owner.email,last_interaction_at
```

Lists prospects with no contact in 2 weeks, narrowed to the fields a re-engagement workflow needs.

### Bulk reassign tasks

```bash
lawmatics-pp-cli bulk reassign --from alice@firm.com --to bob@firm.com --filter matter.status=open --dry-run
```

Preview every task that would move during attorney coverage before committing.

### Revenue by referral source

```bash
lawmatics-pp-cli revenue --by source --window ytd --csv
```

Year-to-date marketing ROI grouped by referral source, ready to drop into a spreadsheet.

### Daily change feed

```bash
lawmatics-pp-cli since 24h --json --select type,id,name,updated_at
```

Standup-ready list of everything that changed yesterday across all resources.

## Auth Setup

Lawmatics uses OAuth 2.0. Run `lawmatics-pp-cli auth login` to open the browser, approve, and the CLI catches the redirect at http://localhost:8765/callback and stores the token in the macOS Keychain. Static bearer tokens are also supported via `LAWMATICS_ACCESS_TOKEN`.

Run `lawmatics-pp-cli doctor` to verify setup.

## Agent Mode

Add `--agent` to any command. Expands to: `--json --compact --no-input --no-color --yes`.

- **Pipeable** — JSON on stdout, errors on stderr
- **Filterable** — `--select` keeps a subset of fields. Dotted paths descend into nested structures; arrays traverse element-wise. Critical for keeping context small on verbose APIs:

  ```bash
  lawmatics-pp-cli addresses list --agent --select id,name,status
  ```
- **Previewable** — `--dry-run` shows the request without sending
- **Offline-friendly** — sync/search commands can use the local SQLite store when available
- **Non-interactive** — never prompts, every input is a flag
- **Explicit retries** — use `--idempotent` only when an already-existing create should count as success, and `--ignore-missing` only when a missing delete target should count as success

### Response envelope

Commands that read from the local store or the API wrap output in a provenance envelope:

```json
{
  "meta": {"source": "live" | "local", "synced_at": "...", "reason": "..."},
  "results": <data>
}
```

Parse `.results` for data and `.meta.source` to know whether it's live or local. A human-readable `N results (live)` summary is printed to stderr only when stdout is a terminal AND no machine-format flag (`--json`, `--csv`, `--compact`, `--quiet`, `--plain`, `--select`) is set — piped/agent consumers and explicit-format runs get pure JSON on stdout.

## Agent Feedback

When you (or the agent) notice something off about this CLI, record it:

```
lawmatics-pp-cli feedback "the --since flag is inclusive but docs say exclusive"
lawmatics-pp-cli feedback --stdin < notes.txt
lawmatics-pp-cli feedback list --json --limit 10
```

Entries are stored locally at `~/.lawmatics-pp-cli/feedback.jsonl`. They are never POSTed unless `LAWMATICS_FEEDBACK_ENDPOINT` is set AND either `--send` is passed or `LAWMATICS_FEEDBACK_AUTO_SEND=true`. Default behavior is local-only.

Write what *surprised* you, not a bug report. Short, specific, one line: that is the part that compounds.

## Output Delivery

Every command accepts `--deliver <sink>`. The output goes to the named sink in addition to (or instead of) stdout, so agents can route command results without hand-piping. Three sinks are supported:

| Sink | Effect |
|------|--------|
| `stdout` | Default; write to stdout only |
| `file:<path>` | Atomically write output to `<path>` (tmp + rename) |
| `webhook:<url>` | POST the output body to the URL (`application/json` or `application/x-ndjson` when `--compact`) |

Unknown schemes are refused with a structured error naming the supported set. Webhook failures return non-zero and log the URL + HTTP status on stderr.

## Named Profiles

A profile is a saved set of flag values, reused across invocations. Use it when a scheduled agent calls the same command every run with the same configuration - HeyGen's "Beacon" pattern.

```
lawmatics-pp-cli profile save briefing --json
lawmatics-pp-cli --profile briefing addresses list
lawmatics-pp-cli profile list --json
lawmatics-pp-cli profile show briefing
lawmatics-pp-cli profile delete briefing --yes
```

Explicit flags always win over profile values; profile values win over defaults. `agent-context` lists all available profiles under `available_profiles` so introspecting agents discover them at runtime.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Usage error (wrong arguments) |
| 3 | Resource not found |
| 4 | Authentication required |
| 5 | API error (upstream issue) |
| 7 | Rate limited (wait and retry) |
| 10 | Config error |

## Argument Parsing

Parse `$ARGUMENTS`:

1. **Empty, `help`, or `--help`** → show `lawmatics-pp-cli --help` output
2. **Starts with `install`** → ends with `mcp` → MCP installation; otherwise → see Prerequisites above
3. **Anything else** → Direct Use (execute as CLI command with `--agent`)

## MCP Server Installation

Install the MCP binary from this CLI's published public-library entry or pre-built release, then register it:

```bash
claude mcp add lawmatics-pp-mcp -- lawmatics-pp-mcp
```

Verify: `claude mcp list`

## Direct Use

1. Check if installed: `which lawmatics-pp-cli`
   If not found, offer to install (see Prerequisites at the top of this skill).
2. Match the user query to the best command from the Unique Capabilities and Command Reference above.
3. Execute with the `--agent` flag:
   ```bash
   lawmatics-pp-cli <command> [subcommand] [args] --agent
   ```
4. If ambiguous, drill into subcommand help: `lawmatics-pp-cli <command> --help`.

---
> Source: [gregvanhorn/lawmatics-pp-cli](https://github.com/gregvanhorn/lawmatics-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
