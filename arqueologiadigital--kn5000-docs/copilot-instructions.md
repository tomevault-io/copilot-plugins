## kn5000-docs

> Jekyll-based documentation website for the KN5000 reverse engineering project. Build: `bundle exec jekyll serve`.

# CLAUDE.md — KN5000 Documentation Website

## Overview

Jekyll-based documentation website for the KN5000 reverse engineering project. Build: `bundle exec jekyll serve`.

## Issue Tracker

Project issues are tracked centrally using [Beads](https://github.com/beads-ai/beads) in `/mnt/shared/kn5000_project/.beads/issues.jsonl`. Use `/mnt/shared/tools/bd` commands (never edit JSONL directly). The `issues.md` page is auto-generated — update via `cd /mnt/shared/kn5000-roms-disasm && make issues`, then commit the result here.

**After meaningful work, agents MUST:** (1) update relevant issues with progress comments, (2) open new issues for next steps, (3) sync issues to this website (`make issues` in roms-disasm), (4) pick the next task from the tracker.

## Policies

- **Keep in sync** with reverse engineering discoveries across all subprojects (see Documentation Freshness policy in central CLAUDE.md).
- **Symbol names in docs must match assembly source** (see Symbol Name Synchronization policy in roms-disasm CLAUDE.md).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArqueologiaDigital)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ArqueologiaDigital)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
