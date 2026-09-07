## trackway

> Instructions for coding agents and people working in this repository.

# Working on Trackway

Instructions for coding agents and people working in this repository.

## What this is

Trackway reads the session files coding agents write to disk and turns them into git-tracked records of decisions, discoveries, questions, and the options that were rejected.

If this repository has a `.trackway/records/` directory, the reasoning behind past choices is in there. Check it before proposing an approach, especially before re-proposing something. `trackway rejected <topic>` lists what was already ruled out and why.

## Layout

Six packages. Dependencies flow one way.

```
cli  ──> distill ──> adapters ──> core
 │                                  ^
 └────> server ─────────────────────┘
            └──> ui (prebuilt, served as static files)
```

- **core** owns the record model, IDs, the file store, the SQLite index, and search. It depends on nothing internal. Anything shared belongs here.
- **adapters** reads sessions. One interface, three implementations, two backing strategies.
- **distill** owns the sweep, the extraction prompt, the headless runner, and the eval harness.
- **server** serves the explorer API and the MCP surface over the same search layer the CLI uses.
- **ui** is the explorer. Built ahead of publishing, never at install time.
- **cli** wires them together and owns nothing else.

## Rules that are not style preferences

These exist because breaking them causes a specific failure. Each is covered by a test.

**Nothing may throw into a coding session.** Trackway runs beside the developer's agent and, with a hook installed, inside its lifecycle. Every entry point returns a fallback. `isolate()` in core exists for this.

**Record identity is content-derived and narrow.** A record's ID hashes its source region, type, and subject, and nothing else. Adding a mutable field to the identity core means superseding a decision changes its own ID and breaks every reference to it.

**Reasoning is stripped at the parse boundary.** Structurally, by block type, before anything reaches disk. Not downstream, not heuristically.

**Redaction is best effort and says so.** Do not tighten a pattern without running it over real transcripts. Every loose version of it produced false positives that synthetic tests missed.

**The index is derived.** It must always be rebuildable from the record files alone. Never make it the only copy of anything.

**Never claim a human approved something.** When an agent decided and proceeded, attribution is `implicit`. Getting this wrong makes the whole store untrustworthy.

**Never truncate a session silently.** Long sessions are chunked with widening windows so coverage stays complete. Truncation was the original approach and it read as complete output while dropping most of a session.

## Adding an agent adapter

Implement `SessionAdapter`: `isAvailable()`, `listSessions()`, `readSession()`, and a capability descriptor. Nothing in core changes.

Two things matter more than they look:

- `listSessions()` runs on every command. Read only the head of a file, never the whole thing.
- Declare `canDistill: false` unless you have actually run that agent non-interactively. An unverified capability fails mid-sweep.

Build a fixture from a real session, anonymized, with a planted reasoning marker and a planted credential. Both existing adapter test suites do this, and both caught bugs that synthetic fixtures did not.

## Commands

```bash
npm install
npm run build      # packages, then the explorer bundle
npm test
npm run typecheck  # strict, covers sources and tests
```

## Measuring extraction

`trackway eval` scores the extractor against sessions that recorded their own option lists. Run it after changing the prompt. It reports and does not gate: suppressing a useful record to protect a score is the wrong trade.

Matching is judged by a model rather than by word overlap. Overlap was tried and cannot recognise a reworded extraction, which is what good extraction produces.

---
> Source: [me-shaon/trackway](https://github.com/me-shaon/trackway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
