## dot-claude-files

> These instructions apply to all projects.

# Global Claude Instructions

These instructions apply to all projects.

## Environment
- Node 20.20.0 / npm 10.8.2 (use nvm if you like. It is available on this machine.)
- Python 3.12 (always work in a project specific virtual environment named .venv at the root of the project.)

## General Preference
- Keep responses concise and direct
- TypeScript: prefer explicit types over `any`
- Python projects: use `src/<package_name>/` layout with relative intra-package imports

## Workflow
- Commit as often as makes sense without asking for permission
- Almost always work in a branch. Use the /branch skill.
- Use PRs for most merges to main. Use the /create-pr and /merge-pr skills.
- Default to targeting `main` as the PR base. If about to target another open branch, pause and confirm — stacked PRs auto-close when the base branch is deleted on merge, which is usually surprising. Prefer merging dependencies in order over stacking.
- Do not push to main without being explicitly asked.
- For documentation-only commits pushed directly to main, add the literal CI-skip directive (square-bracketed "skip ci") to the commit message to avoid burning CI minutes.

## CI-skip directive trap (important)
GitHub Actions parses commit messages for the literal substring `[skip ci]` regardless of context — backticks, code fences, indented blocks, and quoted strings do NOT escape it. If any commit message contains the substring anywhere in any line, GitHub silently skips workflows for that commit. This bites in two specific ways:

1. **Squash-merge contagion**: when you squash-merge a PR, GitHub concatenates every individual commit message into the merge commit. If any sub-commit had the directive (even a documentation-only sub-commit that legitimately wanted to skip CI), the squash commit inherits it and the post-merge run on main is silently skipped. This can leave main in a broken CI state for days without anyone noticing.

2. **Self-quoting bite**: writing about the directive in commit messages or PR descriptions ("merged because [skip ci] was in a sub-commit") triggers the directive on the commit doing the writing.

Mitigations:
- Never write the literal four-character sequence `[skip` followed by `ci]` in any commit message except when you actually intend to skip CI on that exact commit. When you need to refer to it in prose, write "the literal CI-skip directive", "skip-ci marker", or `[skip<space>ci]` with the space spelled out — anything that doesn't match the parser pattern.
- Before merging a PR, check `gh pr checks` reports actual results (not "no checks reported") and that the commit count of completed runs matches what you expect. "No checks reported" can mean "still queueing" OR "skipped via the directive" — always investigate which.
- After merging to main with squash, verify a fresh CI run triggered for the merge commit (`gh run list --branch=main --limit 3`). If none did, the squash inherited a skip directive — the safest fix is a no-op commit to main to re-trigger CI.

## New Repository Setup
- When initialising a new repo, generate an MIT `LICENSE` file with copyright holder "Steven Burns" and the current year.

## File Access
- For large files, prefer Grep (backed by ripgrep) over reading the whole file; use Bash with sed or awk as a fallback when Grep is insufficient.

## Intellectual Honesty
- If a request seems technically wrong, counterproductive, or based on a false assumption, say so before proceeding
- Don't silently comply with an approach that seems like the wrong solution to the actual problem — flag it first
- If there's a clearly better approach than what was asked, mention it even if I didn't ask
- Prefer honest pushback over compliance
- If I've acknowledged a tradeoff and asked you to proceed anyway, do so without further objection

## Project-Specific Notes
See each project's CLAUDE.md for project-specific conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stevenmburns) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
