## claude-for-researchers

> A public guide and toolkit for researchers (especially physicists) who want to

# CLAUDE.md — claude-for-researchers

## What this repo is
A public guide and toolkit for researchers (especially physicists) who want to
use Claude Code productively on long, technically demanding projects. Built from
real experience on a months-long mathematical research project.

## Repo structure
- `README.md` — the main guide (prose + rationale + lessons learned)
- `CLAUDE.md` — this file (instructions for Claude when working on this repo)
- `.claude/skills/` — reusable skill templates users can drop into their own `.claude/skills/`
- `.claude/settings.json` — annotated generic settings + hooks
- `scripts/` — shell scripts for git workflow, GitHub README checks, etc.
- `examples/` — concrete template files (CLAUDE.md template, next-session-prompts, condensed-notes pattern)

## Chat formatting
Plain English in chat. No jargon, no LaTeX. This guide is for people who may
not know Claude Code at all yet, so explanations should be clear to a physicist
who is a fluent programmer but has never used Claude Code.

## Writing style for this repo
- README and example files: friendly but precise. Explain the WHY, not just the HOW.
  Researchers want to understand the reasoning before they trust a workflow.
- Template files: heavily commented, so someone can open them and understand them
  without reading the README first.
- Scripts: POSIX-compatible where possible; add a usage comment at the top of each.
- No overselling. Be honest about what Claude is bad at, what wastes time, and
  what the user should keep control of.

## Key lessons from the source project (inform the guide, don't copy-paste)
1. CLAUDE.md is the most important file. It determines everything about session quality.
   A bad CLAUDE.md costs more time than it saves.
2. The condensed-notes pattern (a short self-contained reference document Claude
   reads first) solves the context-window problem for long projects better than
   trying to make Claude remember things.
3. Skills are powerful but narrow. Write one skill per well-defined task. Don't
   write a "do everything" skill.
4. Hooks solve real problems: pre-commit validation, auto-push to multiple remotes,
   pre-compact state saves. But they are invisible and debugging them is painful —
   document them in CLAUDE.md.
5. GitHub's LaTeX rendering is a strict subset of MathJax. Many standard LaTeX
   commands are blocked. Test README math early.
6. next-session-prompts.md (a human-maintained task log) beats relying on Claude's
   memory across sessions — Claude's auto-memory is good for preferences and facts,
   bad for in-flight research state.
7. Commit discipline matters. Small, atomic commits with clear messages let Claude
   help you recover from mistakes. Large uncommitted working sets do not.
8. The dual-remote pattern (personal GitHub + institution GitLab, different identities)
   is common in academia and needs explicit setup.

## Source project (for reference and examples)
The guide was built from experience on a real research project. That project lives at:

  /Users/mehregandoroudiani/Physics/Eisenstein L-values/

It is a separate git repo (private physics research, NOT part of this public repo).
When working on this guide, Claude can read files from that project using absolute
paths if an example or pattern is needed. Key files there:
- `CLAUDE.md` — a real, production CLAUDE.md (the full template in action)
- `.claude/skills/` — the actual skills used during that project
- `.claude/settings.json` — the actual hooks and permissions in use
- `numerics/` — Python/mpmath computation scripts (usage examples for the guide)
- `condensed.tex` — a real condensed reference document (~26 pp)
- `next-session-prompts.md` — a real session log

Do NOT copy content (formulas, physics results) from that project into this one.
Use it only to check that the guide's advice matches what actually worked.

## Git for this repo
- Remote 'origin': https://github.com/Mexregkan/claude-for-researchers.git
- Push: git push origin main
- Commit author: Mehregan Doroudiani <287984940+mehregandoroudiani-ship-it@users.noreply.github.com>
- No Co-Authored-By trailers.

## What NOT to include
- Anything from the source physics project (content, formulas, results)
- Personal email addresses or account names (use placeholders like YOUR_GITHUB_USER)
- Any claim that Claude is a research collaborator or co-author (it is a tool)

## ⚠️ Auto-saved before context compact [2026-06-05 20:54]
Session was compacted. Last known state is in the "Current status" section above.
To resume: start new session, open next-session-prompts.md, paste the top prompt.

---
> Source: [Mexregkan/claude-for-researchers](https://github.com/Mexregkan/claude-for-researchers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
