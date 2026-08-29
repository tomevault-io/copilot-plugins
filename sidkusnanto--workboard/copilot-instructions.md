## workboard

> This document is for coding agents working on the Workboard codebase.

# AGENTS.md

This document is for coding agents working on the Workboard codebase.

## Project operations

- Do not create Linear issues, stories, or subissues for Workboard unless a future task explicitly asks for them.
- If implementation work changes code, open a GitHub PR after verification.
- If Workboard behavior changes, update the bundled `using-workboard` skill in `skills/using-workboard/` in the same change so agent guidance stays current. Treat `skills/using-workboard/SKILL.md` and `skills/using-workboard/references/current-cli.md` as required touchpoints whenever CLI behavior, file protocol expectations, or supported workflows change.

## Mission

Build a **Go-based**, **cross-platform**, **file-native ticket management system** for software teams and coding agents.

The product is optimized for **agents first** and **humans second**.

The filesystem protocol matters more than UI polish. Preserve the protocol, the invariants, and recovery properties.

## Non-negotiable constraints

1. **Filesystem is the source of truth** in v1.
2. **No required database**.
3. **Cross-platform support is required** for macOS, Windows, and Linux.
4. **Synced-folder safety matters more than real-time behavior**.
5. **Use append-only event files** for shared activity.
6. **Treat file watching as optional optimization**, never a correctness dependency.
7. **Assume consumer sync latency and occasional conflict copies**.
8. **One active implementation claim per ticket should win**.
9. **Queues and indexes are derived state**, never canonical state.
10. **Git remains the system of record for source code changes**.

## Product summary

Workboard is a filesystem protocol plus a CLI/reconciler.

It should allow multiple coding agents to coordinate work by reading and writing plain files inside a shared folder.

The implementation must be easy to inspect, easy to script, and easy to recover after partial corruption or sync weirdness.

## Build priorities

Priority order:

1. stable file protocol
2. validation and recovery
3. claim semantics
4. deterministic indexing
5. CLI usability
6. optional watch mode

Do not optimize for interactivity before the protocol is solid.

## MVP deliverables

Implement these commands first:

- `workboard init`
- `workboard new`
- `workboard show`
- `workboard claim`
- `workboard release`
- `workboard event`
- `workboard index`
- `workboard validate`
- `workboard doctor`
- `workboard reconcile`

### Command expectations

#### `workboard init`
Create a fresh folder scaffold with templates and base config.

#### `workboard new`
Create a new ticket folder with `ticket.md`, `spec.md`, `plan.md`, `result.md`, plus `events/` and `claims/` directories.

#### `workboard show`
Display a ticket summary by reading canonical files.

#### `workboard claim`
Create or replace the current actor's claim file for a ticket. Emit a `claim-acquired` event.

#### `workboard release`
Mark or remove the current actor's claim and emit `claim-released`.

#### `workboard event`
Append a well-formed event file to a ticket.

#### `workboard index`
Rebuild derived indexes and queue files from canonical state.

#### `workboard validate`
Parse and validate canonical files without mutating them.

#### `workboard doctor`
Detect broken invariants and report repair guidance.

#### `workboard reconcile`
Resolve derived state and lightweight conflicts. It may also expire claims and surface contested tickets.

## File protocol invariants

Preserve these invariants.

### Invariant 1: Canonical state is local to the ticket or project root
Canonical files:
- `project.yaml`
- `agents/*.yaml`
- `tickets/*/ticket.md`
- `tickets/*/spec.md`
- `tickets/*/plan.md`
- `tickets/*/result.md`
- `tickets/*/events/*.yaml`
- `tickets/*/claims/*.yaml`

### Invariant 2: Derived state must be safe to delete
Derived files:
- `indexes/*.json`
- `queues/*.txt`

### Invariant 3: Shared activity should prefer new files
When multiple actors may act concurrently, prefer creating a new event file rather than editing a shared file.

### Invariant 4: Claim conflicts must be detectable and resolvable
If multiple active implementation claims exist, the reconciler must determine the winner deterministically and flag the ticket as contested.

### Invariant 5: Ticket summary files stay compact
`ticket.md` should remain easy for an agent to read quickly.

## Deterministic conflict policy

When two active implementation claims exist for the same ticket:
- earliest unexpired claim wins
- tie-break by lexical `actor_id`
- both claims remain visible
- reconciler records the contest in derived state or emits a warning
- no data is silently discarded

## Suggested repository layout

```text
/workboard-cli/
  cmd/
    workboard/
      main.go
  internal/
    app/
      app.go
    cli/
      init.go
      new.go
      show.go
      claim.go
      release.go
      event.go
      index.go
      validate.go
      doctor.go
      reconcile.go
    model/
      project.go
      agent.go
      ticket.go
      claim.go
      event.go
      index.go
    parser/
      frontmatter.go
      markdown.go
      yaml.go
      time.go
    fsstore/
      workboard.go
      tickets.go
      claims.go
      events.go
      atomic.go
      paths.go
    reconcile/
      claims.go
      status.go
      queues.go
      indexes.go
    validate/
      schema.go
      invariants.go
    templates/
      templates.go
  templates/
    project.yaml
    ticket.md
    spec.md
    plan.md
    result.md
    agent.yaml
  testdata/
    valid/
    invalid/
  README.md
  AGENTS.md
```

This layout is a suggestion, not a strict requirement. The important part is to separate:
- domain models
- parsing/serialization
- filesystem storage
- reconciliation logic
- CLI entry points

## Data model expectations

### Ticket
A ticket model should capture:
- id
- title
- status
- priority
- timestamps
- requester
- owner
- labels
- dependencies
- references to spec/plan/result
- summary and acceptance criteria extracted from markdown if needed

### Claim
A claim model should capture:
- claim id
- ticket id
- actor id
- actor type
- acquired at
- expires at
- intent
- status

### Event
An event model should capture:
- event id
- ticket id
- timestamp
- actor
- kind
- summary
- payload

### Project config
Project config should capture:
- version
- project id
- status list
- priority list
- claim policy
- write policy
- index settings

## Implementation rules

### Path handling
Use `filepath` utilities consistently. Never assume slash-separated paths in logic.

### File writes
Use careful write patterns.

Preferred approach:
1. serialize to memory
2. write temp file in same directory if useful
3. rename into place when safe

However, remember that some sync providers still treat rename operations awkwardly. For shared activity, a new event file is usually safer than replacing a shared file.

### Time format
Use UTC timestamps in filenames and machine-readable fields.

### ID format
Use a stable unique ID format for events and claims. Human-friendly ticket IDs may be assigned differently from opaque event IDs.

### Watching
If a watch mode exists, it must only trigger scans/reconciliation. It must not be the only way the system stays correct.

### Logging
Keep logs human-readable. Avoid noisy structured logs in normal CLI output. Machine-readable output can be added later if useful.

## Testing expectations

Build a strong file-based test suite.

Required categories:
- scaffold creation
- frontmatter parsing
- YAML parsing
- invalid schema detection
- index rebuild from canonical files
- claim conflict resolution
- expired claim handling
- CRLF/LF compatibility where practical
- path handling on Windows-like paths where possible

Also include golden-file style tests for generated markdown, YAML, and index outputs.

## First milestone plan

### Milestone 1: scaffold and parsing
Deliver:
- project scaffold templates
- markdown frontmatter parser
- YAML parser helpers
- domain models
- `init` and `validate`

Done when:
- a new workboard folder can be created
- canonical files round-trip cleanly
- malformed files produce clear validation errors

### Milestone 2: ticket lifecycle basics
Deliver:
- `new`
- `show`
- template rendering
- minimal ticket loading

Done when:
- a ticket can be created and displayed from disk

### Milestone 3: claims and events
Deliver:
- `claim`
- `release`
- `event`
- event filename generation
- claim expiry logic

Done when:
- an actor can claim a ticket
- activity is appended as event files
- conflicting claims are detectable

### Milestone 4: indexing and reconciliation
Deliver:
- `index`
- `doctor`
- `reconcile`
- derived queue generation
- open claims index
- agent pick list index

Done when:
- derived state can be fully regenerated from canonical files
- contested tickets and expired claims are surfaced

### Milestone 5: hardening
Deliver:
- better error messages
- more tests
- platform compatibility fixes
- optional watch-triggered scan mode if still useful

Done when:
- the CLI feels stable on all target platforms

## Suggested initial derived indexes

Implement these first:
- `indexes/tickets_by_status.json`
- `indexes/open_claims.json`
- `indexes/agent_picklist.json`

Example `agent_picklist.json` shape:

```json
{
  "generated_at": "2026-03-12T18:30:00Z",
  "ready_unclaimed": [
    {
      "id": "T-0001",
      "priority": "P1",
      "title": "Add passwordless email login"
    }
  ],
  "blocked_needing_human": [
    {
      "id": "T-0006",
      "reason": "Need API key"
    }
  ]
}
```

## Required ticket templates

At minimum, create templates for:
- `ticket.md`
- `spec.md`
- `plan.md`
- `result.md`
- `project.yaml`
- `agent.yaml`

Keep the templates small and agent-readable.

## UX guidance

Humans are secondary, but not irrelevant.

The CLI should:
- print concise output
- fail loudly and specifically
- never silently repair canonical data without making that obvious
- make it easy to understand what file changed

## Things to avoid

Do not:
- introduce a required database
- store canonical state only in generated indexes
- depend on OS-specific file locking semantics
- assume sync order is immediate or correct
- make watch mode mandatory
- overfit the design to one sync provider
- let multiple agents mutate the same ticket summary in hot loops

## Stretch goals after MVP

Only consider these after the core protocol is solid:
- TUI
- web UI
- JSON output mode for automation
- Git integration helpers
- PR summary generation
- approval workflow helpers
- local daemon mode

## Definition of done for the handoff

A good first implementation is one where another agent or human can:
- initialize a workboard
- create tickets
- claim and release work safely
- append events
- regenerate indexes from canonical files alone
- validate broken states
- understand the system by reading the folder structure and docs

## Working style for the coding agent

When making implementation decisions:
- choose boring, reliable approaches first
- prefer protocol stability over clever abstractions
- preserve recovery properties
- write tests for filesystem edge cases early
- keep the code easy for later agents to inspect and extend

If a choice is unclear, prefer the option that makes the on-disk state simpler, more explicit, and easier to reconstruct.

---
> Source: [sidkusnanto/workboard](https://github.com/sidkusnanto/workboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
