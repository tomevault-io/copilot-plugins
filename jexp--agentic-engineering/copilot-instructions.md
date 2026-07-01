## agentic-engineering

> Working repo for two upcoming sessions:

# CLAUDE.md — Agentic Engineering workshop & keynote prep

## What this is
Working repo for two upcoming sessions:
- **Workshop** (4 hours) — *next day*: experienced engineers / team leads, hands-on agentic engineering
- **Keynote** (~30 min) — *2 weeks*: VibeCode conference, "From Vibe Coding to Agentic Engineering"

**Public repo**: this folder will be published at **github.com/jexp/agentic-engineering**. Treat all docs as public-facing — no NDA material, no internal codenames, no leaked-source provenance, no internal infra paths.

## Conventions for working here

- **Working language**: English. Markdown only — no slide-builder yet.
- **Tone**: tight, opinionated, written for experienced engineers who've seen the hype. Avoid AI-flavored padding ("In today's rapidly evolving landscape…"). Lead with the claim, then back it up.
- **Slide drafts**: structure as `## Slide N — Title` with talking-point bullets, not paragraphs. The user will turn these into slides separately.
- **Quotes**: tag every quote with `[VERIFY]` unless we're certain of the wording. Never invent quotes.
- **Internal Anthropic codenames**: keep entirely out of public docs and slides — this repo is published at github.com/jexp/agentic-engineering. Use generic descriptions ("fast tier", "remote planning mode") instead. Reserve codename color for private Q&A only.
- **Numbers** (benchmark scores, prices): tag with `[VERIFY]` and a date. Pricing and benchmark leaderboards move monthly.

## Logging — always update both

- **[`progress.md`](./progress.md)** — running log. Every new artifact: add a file entry, a Done bullet (with date), and update Next Up. Don't ask before logging.
- Per-section docs ship in their own `workshop-<topic>.md` file. Each ends with an "Open questions for me to decide before tomorrow" list.

## Repo layout (post-restructure 2026-05-04)

```
README.md           — public-facing entry
CLAUDE.md           — this file
progress.md         — running log

workshop/           — 4hr workshop materials
  workshop-outline.md          (master running order — start here)
  workshop-intro.md
  workshop-exercise-1.md
  workshop-harnesses.md
  workshop-tool-use.md
  workshop-cost-tokens.md
  workshop-benchmarks.md
  workshop-practices.md
  workshop-own-experience.md
  workshop-mental-safety.md
  workshop-exercise-2.md
  workshop-resources.md
  agentic-engineering-ideas.md (raw notes)

keynote/            — keynote drafts
  prompt.md (original brief)
  vibecode-keynote-v{1,2,3}.md

research/           — source material that fed the workshop
  coding-agents/                       (16 per-tool deep dives + README)
  context-engineering-mining.md
  context-engineering-presentation.pdf
  agent-hooks.md
  github-growth.txt

assets/             — images, QR codes
  qrcode-github-jexp-agentic-engineering.png
```

## Reference material outside this repo (don't re-derive — read)

- `~/d/llm/claude-code/` — deep CC source-level analysis (architecture.md, findings.md, tools.md). The canonical reference for harness anatomy.
- `~/d/llm/<projects>/` — own-experience demos (sather-k, vibe-haskell, neo4j-skills, claude-code analysis, openalex-neo4j, clauditopia, inferrs, aura-cli, agent-intelligence, aura-agent-memory-gateway).
- `~/Downloads/` — PDFs, skill files, additional analysis material.
- User's `~/.claude/CLAUDE.md` — global preferences; `~/.claude/RTK.md` — RTK reference.

## What to do when adding a new section

1. New file at `workshop-<topic>.md`. Use the established structure: framing line → numbered slides → sources/references → open questions.
2. Update [`progress.md`](./progress.md) — add a Files entry and a Done bullet (datestamp + ~3-line summary of what's in the doc).
3. End the section with explicit open questions tagged `- [ ]` so the user can decide before stage time.

## Things that have been agreed

- **Own-experience section is demo-only** — slides are cue cards; full live demos against real folders.
- **Codenames stay off slide-style lines** — keep generic anatomy public-facing.
- **Quotes tagged [VERIFY]** until confirmed wording.
- **Lean towards naming specific harnesses/numbers** — anonymity reads as evasion to engineers.

## Don't

- Don't auto-commit; never run git commit without asking.
- Don't invent quotes or benchmark numbers.
- Don't pad slides with generic AI commentary; the audience can smell it.
- Don't add features beyond what the user asked. If a section is "3 slides," don't ship 6.

---
> Source: [jexp/agentic-engineering](https://github.com/jexp/agentic-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
