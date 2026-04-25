## parseclipmate

> **Zgent Status:** zgent (in-process toward Zgent certification)

# ParseClipmate — Clipboard Ingestion and Processing Pipeline

**Zgent Status:** zgent (in-process toward Zgent certification)
**Role:** Consumer — clipboard content ingestion and processing
**Bead Prefix:** `pc`

## STOP — Beads Gate (Read This First)

**This repo is beads-first. You MUST authorize work before doing it.**

Before making ANY substantive changes (creating/modifying files, installing deps, changing config), do this:

```bash
bd ready                    # See if there is already an open bead for this work
bd create -t "Short title"  # Create one if not — YOU own this, do not ask the user
bd update <id> --status in_progress  # Claim it
```

When done:
```bash
bd close <id>               # Mark complete
bd sync                     # Sync with git
```

Reference the bead ID in your commit messages: `[pc-xxx] description`.

**No bead = no work.** Minor housekeeping (typos, status fields) is exempt. Everything else gets a bead. If in doubt, create one — it is cheap. See `.claude/rules/beads-first.md` for the full rule.

**This is not optional. This is not a Gas Town thing. This is how THIS repo works, every session, every instance.**

## What This Is

ParseClipmate is the enterprise clipboard ingestion zgent. It captures, parses, and processes clipboard content for use across the Gas Town enterprise.

> **Recovery**: Run `gt prime` after compaction, clear, or new session. Full context is injected by `gt prime` at session start.

## What Every Claude Instance Must Understand

1. **Beads-first is non-negotiable.** Read the gate at the top of this file. Use `bd` commands. No exceptions.
2. **Consumer permissions.** Standard zgent access — read enterprise, write own repo. See `.claude/rules/zgent-permissions.md`.

## Graduation Status

- **Standard artifacts deployed** — beads-first, zgent-permissions, settings.json, .gitattributes ✓

## Conventions

- Beads-first: self-bead for non-trivial work, reference bead ID in commits
- Enterprise permissions: read sibling repos, write only own path

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justSteve) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
