## bdboard

> This project uses **bd** (beads) for issue tracking. Run `bd prime` for full workflow context.

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd prime` for full workflow context.

> **Architecture in one line:** Issues live in a Dolt database (`.beads/`).
> Bead data is replicated off-machine via Dolt's git-compatible wire protocol:
> the Dolt remote `origin` points at the same GitHub origin as the code, and
> issue history rides under `refs/dolt/data` there. After committing local `bd`
> writes, run `bd dolt push` to sync; `bd dolt pull` (or `bd bootstrap` on a
> fresh clone) hydrates from the remote. `.beads/issues.jsonl` is a passive
> export, not the wire protocol.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
```

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

## Critical Don'ts (read before you do anything)

These are hard-won lessons. Each was learned the painful way. Run `bd memories` at session start for the full list of operational gotchas; the rules below are the ones serious enough to live here permanently.

- **Don't write local session summaries.** Never create `COMPLETION_SUMMARY_*.md`, `VERIFICATION_*.md`, `NOTES_*.md`, or similar files anywhere in the repo unless the user **explicitly asks for one**. The bead is the canonical record — capture verification, evidence, snapshots, contrast calculations, and reasoning via `bd update <id> --append-notes "..."`. Root-level (and stray `notes/`) summary files are agent panic-litter and will be deleted.
- **Don't close beads.** The LLM judge pipeline is the only legitimate closer. If you believe a bead is complete, leave it `in_progress` and document evidence in `--append-notes`. Closing yourself bypasses verification and tends to trigger fallback behaviors (like writing summary files) that pollute the repo. The user may explicitly authorize you to close a specific bead per-request; that authorization does not extend to others.
- **Don't run `bd edit`.** It opens `$EDITOR` and hangs the agent indefinitely. Use `bd update --description / --notes / --append-notes / --design` for content edits, and `bd update --claim / --status` for state changes.
- **Don't trust `--dry-run` with `bd create --graph`.** It is silently ignored (bd 1.0.3) and issues are created anyway. Validate plans by hand-reading the JSON and running `jq . plan.json` before applying.
- **Don't expand scope silently.** If you find an unrelated bug while working a bead, follow the Bug Discovery Protocol below — file it; only fix inline if it BLOCKS your current bead.
- **Don't reinvent prior art.** Run `bd memories` and `bd memories <keyword>` at session start. Search beads with `bd search <topic>` before filing new ones. Many "obvious" patterns and gotchas are already documented.
- **Don't display raw `dependency_type` as the relationship label.** bd reports `"blocks"` on both sides of a blocks edge; the label depends on direction AND type. See memory `dep-edge-direction` and bdboard-fjk for the full mapping.

## Bug Discovery Protocol

When you find a bug **unrelated to your current bead's stated goal**:

**One bug per bead.** Multiple unrelated issues → file each separately.

**Decide BLOCKING vs NON-BLOCKING per bug:**
- **BLOCKING** = you cannot satisfy your current bead's acceptance criteria without fixing the bug first.
- **NON-BLOCKING** = the bug exists but doesn't prevent you from completing the current bead.

**NON-BLOCKING — file and keep working:**
```bash
bd create --type=bug --priority=2 \
  --title="<short title>" \
  --description="<what you saw, repro steps, suspected cause>"
```
Then continue with your original bead.

**BLOCKING — file with triage marker, fix inline, finish work:**
```bash
# 1. File the bug with a discovered-from edge (NOT blocks).
#    Use discovered-from because after you fix inline the current bead
#    is no longer waiting on it — the bug bead exists for separate
#    verification later. A `blocks` edge here creates a self-deadlock:
#    the current bead can't close because the (intentionally) still-open
#    bug remains a blocker. discovered-from preserves traceability without
#    blocking close. (Learned the hard way via bdboard-lng ↔ bdboard-3y7.)
bd create --type=bug --priority=1 \
  --title="<short title>" \
  --description="[bead-chain:triaged] <what you saw, what you fixed inline, why it blocked>"
# Capture the returned id as <new-bug-id>, then:
bd dep add <current-bead-id> <new-bug-id> -t discovered-from
```
Then fix the bug as part of this bead's work (scope expansion), finish the original goal, and present both in your summary so the judges see the expanded scope. The filed bug stays open for a future iteration's proper verification — that's intentional, not a defect.

> **DO NOT** use `--blocks=<current-bead-id>` for triaged-and-fixed-inline bugs. That edge persists after you finish, and `bd close` will refuse to close your bead because the (intentionally) still-open bug remains a blocker. Use `discovered-from` instead.

**The bug-discovery protocol is about *filing*, not closing.** Do NOT close any bead yourself (see Critical Don'ts).

## Bead Artifact SOP

When filing or working ANY bead (bug, task, feature, spike, decision, chore, etc.) that has attached evidence — screenshots, mockups, logs, traces, repro recordings, diagrams, design exports, data dumps:

1. Create (or identify) the bead first — you need the bead ID for the directory name.
2. `mkdir -p notes/<type>/<bead-id>/` where `<type>` matches the bead category:
   - bugs → `notes/bugs/<bead-id>/`
   - tasks / features / chores / spikes / decisions / stories → `notes/tasks/<bead-id>/`
   (If your project already groups artifacts under a single dir, follow the existing convention — the invariant is one subdir per bead ID, never loose files.)
3. Move artifacts into that dir using non-interactive flags: `mv -f <artifact> notes/<type>/<bead-id>/`
   - Rename cryptic/auto-generated filenames (e.g. `Screenshot 2026-... PM.png`) to something descriptive (`masthead-light.png`). Avoid spaces and non-breaking spaces in filenames — they break tooling and image loaders.
4. Reference every artifact path explicitly in the bead's notes:
   ```bash
   bd update <bead-id> --append-notes "Artifact: notes/<type>/<bead-id>/screenshot.png"
   ```
5. `git add notes/<type>/<bead-id>/` and commit alongside the bead filing/update.

Do NOT leave artifacts at the repo root, in `notes/`, or directly in `notes/bugs/` / `notes/tasks/` (i.e. NOT inside the category dir but outside a bead-ID subdir) — they get orphaned from their bead.

## Tool Conventions (summary)

- **Persistent knowledge** → `bd remember "<insight>" --key <short-key>` (injects at `bd prime`).
- **Issue tracking** → `bd` (`bd create`, `bd update`, `bd ready`). NOT TodoWrite, TaskCreate, markdown TODO lists, or comment-based TODOs in code.
- **Verification evidence** → bead `--append-notes`. NOT local summary files.
- **Multi-bead plans** → `bd create --graph plan.json` (no `--dry-run`; validate with `jq` first).

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:7510c1e2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a Dolt DB; bead data is replicated off-machine via Dolt's git-compatible wire protocol (`bd dolt push`/`pull` to `refs/dolt/data` on the same GitHub origin as the code); `.beads/issues.jsonl` is a passive export.

## Session Completion

**When ending a work session**, complete the steps below. Bead data syncs
off-machine via Dolt, so finishing a session means committing code locally
**and** pushing bead writes to the remote with `bd dolt push`.

**WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **COMMIT LOCALLY** - Commit all changes:
   ```bash
   git add -A
   git commit -m "<message>"
   git status  # MUST show "nothing to commit, working tree clean"
   ```
5. **PUSH BEAD DATA** - Replicate bead writes off-machine:
   ```bash
   bd dolt push   # syncs issue history to refs/dolt/data on origin
   ```
6. **Clean up** - Clear stashes
7. **Verify** - All changes committed and bead data pushed
8. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Commit code locally AND `bd dolt push` bead writes — both are needed to sync work off-machine
- NEVER leave changes uncommitted - that leaves work stranded in the working tree
- If a commit fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->

---
> Source: [weegens-aaron/bdboard](https://github.com/weegens-aaron/bdboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
