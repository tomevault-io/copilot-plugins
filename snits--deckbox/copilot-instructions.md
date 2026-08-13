## deckbox

> - User count and tool type: A small number of developers maintain a browser-based tabletop deck-definition editor and its supporting Rust workspace.

# Agent Instructions

## PROJECT SCALE CONTEXT

- User count and tool type: A small number of developers maintain a browser-based tabletop deck-definition editor and its supporting Rust workspace.
- Codebase size and complexity preferences: The codebase is small to medium; prefer straightforward, domain-focused changes over framework or subsystem expansion.
- Process overhead expectations: Use focused tests and checks for each web UI change, while keeping issue tracking and verification evidence explicit.
- Default approach: Pragmatic delivery with maintainable public interfaces and real component/store behavior tests.

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
bd sync               # Sync with git
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

<!-- BEGIN KATA (managed by `kata init --with-agents`) -->
## kata issue tracker

This project uses [kata](https://github.com/kenn-io/kata) as its shared issue
ledger. Run `kata quickstart` at the start of each session for the full agent
contract. The short version:

- Search before creating: `kata search "<keywords>" --agent`.
- Prefer updating existing issues over duplicates (`kata comment`, `kata label add`, `kata edit`).
- Default to `--agent` for ordinary reads and mutations; use `--json` only when a script needs structured data.
- Close only verified work: `kata close <ref> --done --message "<scope + verification>" --commit <sha>`.
- If work is incomplete, label `needs-review` and comment what remains rather than closing.
- Never `kata delete` or `kata purge` without explicit user authorization.

## kata work.* conventions (agent orchestration)

When working a kata-tracked issue, keep its `work.*` metadata truthful
(see docs/operations/agent-orchestration.md for the full recipe):

- On claim/start: `kata meta set <ref> work.attention ok`; if the work has a
  dedicated branch, stamp it once with `kata meta set <ref> work.branch <branch>`.
- Signal live state: `kata meta set <ref> work.attention stuck|needs-human|ok`
  plus a one-line `work.attention_msg` saying why. Raise `stuck` when you cannot
  proceed, `needs-human` when you want review; clear back to `ok` when unblocked.
- Never stop with the signal stale: close the issue, or leave the attention
  pair reflecting the hand-off.
- Coordinators read `work.*` on issues they delegated; only the working agent
  writes them. `work.*` on closed issues is meaningless.
<!-- END KATA -->

---
> Source: [snits/deckbox](https://github.com/snits/deckbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
