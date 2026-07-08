## prismalens-io

> Instructions for AI coding agents working in this repository.

# AGENTS.md

Instructions for AI coding agents working in this repository.

<!-- BEGIN mage -->
## mage knowledge base (hub-linked)

This repository's durable knowledge lives in a **mage hub** at
`/home/sumit/prismalens-org/prismalens-docs-hub`, where this repo is the **prismalens-io** project. mage is a portable,
file-based knowledge base of notes — insight, procedure, and pointers (not
copies of sources) — navigable as an Obsidian graph.

**Before non-trivial work in this repo:**

1. Read the hub index first: `/home/sumit/prismalens-org/prismalens-docs-hub/INDEX.md` — find the **prismalens-io** wing (its
   notes are grouped there; in a large hub the wing links out to its own
   `/home/sumit/prismalens-org/prismalens-docs-hub/_index.prismalens-io.md`). One line per note: type · title · keywords · → link. Open
   only the notes the task touches; don't read everything.
2. Skim `/home/sumit/prismalens-org/prismalens-docs-hub/decisions/` for the hub's governing decisions.
3. Treat notes as point-in-time. If a note is `status: stale-suspect`, or its
   `last_reviewed` / `provenance.commit` looks old, verify it against the
   current code before relying on it.

**After you learn something durable** — an interface detail, a gotcha, how two
services couple, a faster path to a source — capture it with `mage:learn` into
the hub. Capture the reusable *insight + procedure + pointers*, never a copy.

**Capture lessons inline, at first sight.** When you learn something durable
mid-task, stage a SHORT draft right then — `mage stage --title "..." --tags
wing/room` (body on stdin; it is scrubbed and parked in `.staging/`). No per-note
confirm; you batch-review the drafts later with `mage:groom`. Don't wait for a
session boundary — capture at first sight.

**Commit hygiene:** mage never commits for you. It suggests `git` commands; you
run them.
<!-- END mage -->

---
> Source: [prismalens/prismalens.io](https://github.com/prismalens/prismalens.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
