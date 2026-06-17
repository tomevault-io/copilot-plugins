## ai-skills

> This file tells an AI agent (Claude Code, Codex, Cursor, Aider, Gemini CLI, or any other harness) what this repository is and how to install its skills.

# AGENTS.md — for arbitrary AI agents landing in this repo

This file tells an AI agent (Claude Code, Codex, Cursor, Aider, Gemini CLI, or any other harness) what this repository is and how to install its skills.

## What this repo is

A collection of **Anthropic-format skills** for academic research and writing, scoped to the needs of computer science / mathematics students and professors. The skills cover the full research-to-publication pipeline:

- `pull` — fetch external documents (webpages, PDFs, papers) into a local `sources/` folder as Markdown with provenance frontmatter.
- `ingest` — compile pulled sources into a cross-linked wiki at `wiki/`.
- `distill` — extract atomic claims (Zettelkasten-style) from sources into `notes/atomic/`.
- `sota` — produce a structured state-of-the-art review organized by user-relevant dimensions (paradigms, techniques, evaluation methods, domains), output to `sota/`.
- `review` — forensic audit of an academic document (thesis or paper); produces a narrative report (5–18 pages by level) plus a structured forensic assembly sibling.

All five skills ship in this repo under `skills/`:

```
skills/
├── pull/          (source-pulling pipeline via markitdown)
├── ingest/        (sources → cross-linked wiki)
├── distill/       (sources → atomic Zettel notes)
├── sota/          (corpus → structured state-of-the-art review)
└── review/        (academic document → narrative audit + forensic assembly)
```

Each is self-contained, ported with vault-specific paths removed and defaults set to the current working directory.

## How to install the skills (the action items for an agent)

The natural unit of action when an agent reads this repo is: **copy each subdirectory of `skills/` into the agent's skills directory**. Then the agent can invoke them by name (`/sota`, `/review`, etc.).

### Standard paths

| Agent | Skills directory |
|---|---|
| Claude Code (user-global) | `~/.claude/skills/` |
| Claude Code (project-local) | `<project>/.claude/skills/` |
| Codex | `~/.codex/skills/` *(check Codex docs for latest convention)* |
| Cursor | inspect Cursor's settings for skill / rule directory |
| Gemini CLI | depends on harness configuration |
| Generic | wherever the harness expects skill packages |

### The fast path: `install.sh`

If the harness has Bash and the agent can run shell commands, the cleanest install is:

```bash
./install.sh
```

This copies every subdir of `skills/` into `${SKILLS_DEST:-$HOME/.claude/skills/}`. Override the destination via env var:

```bash
SKILLS_DEST=$HOME/.codex/skills ./install.sh        # user-global, Codex
SKILLS_DEST=./.claude/skills ./install.sh           # project-local, Claude Code
SKILLS_DEST=./mi-proyecto/skills ./install.sh       # arbitrary
```

By default `install.sh` skips skills that already exist at the destination (does not overwrite). Pass `--force` to re-install over existing copies.

### The manual path: `cp`

If shell execution is not available or the agent prefers to act file-by-file:

```bash
mkdir -p ~/.claude/skills/
cp -r skills/sota         ~/.claude/skills/
cp -r skills/review       ~/.claude/skills/
# ... etc.
```

Each skill subdir is self-contained: it has its own `SKILL.md` (entry point for the main agent), optionally a `tools/` subdir with vendored scripts, and subagent prompt files (`synthesizer.md`, `assembler.md`, etc.) as siblings.

### Verifying install

After install, restart the agent session (or use the harness's "reload skills" mechanism if available). The skill descriptions should appear in the available-skills list, and invocations like `/sota <topic>` should resolve.

## Skill anatomy (so the agent can verify integrity)

Every skill in this repo follows the same shape:

```
skills/<name>/
├── SKILL.md           ← REQUIRED. YAML frontmatter with `name:` + `description:`,
│                        followed by the main agent prompt body.
├── synthesizer.md     ← optional, for skills with parallel-subagent phases
├── assembler.md       ← optional, for skills with a final synthesis step
├── (other .md)        ← additional subagent prompts as needed
└── tools/             ← optional, for vendored deterministic scripts
    └── *.py / *.sh
```

The `SKILL.md` YAML frontmatter must include `name:` (matching the directory name) and `description:` (telling the harness when to invoke). Both are surfaced in the harness's available-skills list at session start.

`SKILL.md` body is in **English**. Output artifacts (reports, narratives, observations) are produced in the language of the corpus being processed or in the user's interaction language; the harness handles the multilingual routing.

## What the skills assume about the workspace

Default output convention (relative to the agent's current working directory):

```
<cwd>/sources/        ← raw external documents (pull writes here)
<cwd>/wiki/           ← cross-linked synthesis (ingest writes here)
<cwd>/notes/atomic/   ← atomic-claim notes (distill writes here)
<cwd>/sota/           ← state-of-the-art reports (sota writes here)
<cwd>/reviews/        ← document audits (review writes here)
```

Every skill resolves its corpus/output paths in this order:

1. Explicit user-specified path in the invocation.
2. Conventional subdir under `<cwd>` (above).
3. Fallback to `<cwd>` itself.

No skill assumes Obsidian, Notion, a vault, or a git remote. The repo and the skills are designed to work in any folder with Markdown files.

## What the skills DO NOT do

- They never delete or overwrite files without an explicit `--force` style override.
- They never commit or push to git on the user's behalf.
- They never call external services with the user's API keys without surfacing the cost; web fetches via `pull` are explicit, single, and bounded.
- They never produce verdicts of the form "this is good" or "this is bad"; outputs are descriptive ("this claim is here, the evidence cited is there, this gap exists"). The reader interprets.

## Conventions an agent should follow when using these skills

1. **Plan first.** Every skill in this repo follows a plan-first protocol: present a plan derived from the actual corpus / document, wait for the user to approve or modify in natural language, then fan out. Do not skip the plan step.
2. **Single level of indirection.** Subagents dispatched by a skill do not dispatch sub-subagents. Subagent parallelism is at the tool level (Read/Grep/WebSearch batched within one turn).
3. **Multilingual output.** Skill prompts and `SKILL.md` body are in English. Artifacts produced (findings, observations, reports) are in the language of the corpus or document. Frontmatter field names and ID prefixes (`F-V<n>-NNN`, `D<n>`, `[Author Year]`) stay in English.
4. **Markdown is canonical.** Never produce HTML, Word, or PDF as the source of truth. Optional renderings (via Quarto or Pandoc) are companions, never replacements.
5. **No fabricated citations.** When a citation cannot be verified, mark it `(metadata incomplete)` in the bibliography. Never invent an arXiv ID, DOI, or venue.

## Where to learn more

- Each skill's `SKILL.md` is self-documenting; start there.
- `README.md` is human-facing (students and professors).
- This file (`AGENTS.md`) is agent-facing.
- Issues, requests for new skills, and bug reports: GitHub issues on this repo.

## Upstream

These skills are extracted and adapted from a personal Claude Code workspace where they emerged organically across 2025–2026 in support of teaching, research, and writing. They are released to MatCom under MIT for community use. Upstream maintainer: `@apiad` (Alejandro Piad Morffis).

---
> Source: [matcom/ai-skills](https://github.com/matcom/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
