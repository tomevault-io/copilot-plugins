## newsletter-engine

> A repo-based writing system for creating blog and newsletter content. The workflow runs from durable repo files rather than chat memory, coordinating specialised agents across a repeatable content pipeline.

# Newsletter Engine

A repo-based writing system for creating blog and newsletter content. The workflow runs from durable repo files rather than chat memory, coordinating specialised agents across a repeatable content pipeline.

**Primary user:** Jose

---

## Architectural Principles

These govern every skill and orchestration design decision:

1. **File-based I/O contracts** — each skill reads from and writes to a predictable set of files. No skill reaches outside its contract. This is what makes both independent and orchestrated invocation possible.
2. **`post.yaml` is the nervous system** — it is the shared state that every skill reads from and appends to. It is not just metadata — it is the message bus.
3. **Every skill is standalone first** — skills like `/seo` and `/promote` must work on any draft, whether produced by the pipeline or written by Jose independently. No skill should require the full pipeline to have run first.
4. **The orchestrator grows incrementally** — `/new-post` chains only what is available at any point. Each milestone extends it. Separation from the individual skills is maintained throughout.
5. **Separation of concerns** — each skill has one job; the orchestrator has one job (sequencing). This enables triggering any step independently or running the full pipeline end-to-end unattended.

---

## What Is Portable Across Agents

These parts of the system are provider-agnostic and should be treated as the real engine:

- The repo layout and file contracts
- `post.yaml` as the shared state and stage ledger
- The artefact files each skill reads and writes
- The style guides, templates, and reference posts
- The procedures written in `.claude/skills/*/SKILL.md`
- The reviewer personas written in `.claude/agents/*.md`

If an agent can read markdown files and update repo files, it can operate this system.

---

## What Is Provider-Specific

These parts are convenience wrappers, not the core workflow:

- Claude slash-command invocation such as `/draft` or `/review`
- Claude-native agent spawning and parallel sub-agent execution
- Claude-specific MCPs, hooks, and session ergonomics
- Any instruction that assumes a Claude Code-only tool exists

If a provider lacks one of these capabilities, preserve the intended outcome and execute the same file contract manually.

---

## Canonical File Contracts

For any agent working in this repo, the source of truth is:

- `AGENTS.md` — top-level operating manual
- `.claude/skills/*/SKILL.md` — executable procedures for each workflow stage
- `.claude/agents/*.md` — specialist review personas, mainly used by `/review`
- `post.yaml` — shared post state, stage completion, metadata, and artefact pointers
- `templates/`, `style_guide/`, and `reference_posts/` — writing constraints and calibration context
- `posts/<slug>/` — the working directory for each post and all generated artefacts

Agents should prefer updating durable files over returning chat-only results.

---

## How Non-Claude Agents Should Interpret This Repo

If you are not running inside Claude Code:

- Treat each `.claude/skills/*/SKILL.md` as a procedure to execute directly
- Treat each `.claude/agents/*.md` as a reusable role prompt or review persona
- Use the file inputs and outputs described by each skill as the contract to follow
- Respect stage guards and overwrite checks described in the skill before writing files
- Update `post.yaml` whenever a skill says to mark a stage complete or register an artefact

The repo structure matters more than the runtime. If the files are updated correctly, the workflow is considered valid.

---

## Fallback Behavior When Native Tools Do Not Exist

Use these defaults when a provider lacks Claude-specific runtime features:

- No slash commands: open the corresponding `SKILL.md` and execute it manually
- No sub-agent primitive: use `.claude/agents/*.md` as role instructions and run them in the main session
- No parallel agent execution: run critic roles sequentially, then synthesize the results
- No Claude MCP equivalent: continue with local repo files unless the skill truly requires external research
- No hook system: perform the required file updates directly if the workflow depends on them

Do not stop just because a Claude-native convenience is missing. Fall back to the file contract and continue.

---

## Repo Index

| Directory | Purpose |
|-----------|---------|
| `reference_posts/` | Jose's real posts (series, standalone, short_technical) |
| `style_guide/` | Voice, anti-patterns (shared/), per-type rules, promotion_formats.md |
| `.claude/skills/` | Skill instruction files (one per skill) |
| `.claude/agents/` | Critic agent definitions invoked by `/review` (voice, structure, impact) |
| `.claude/hooks/` | Automation hooks: skill-reflector (reflection log), detect-skill-complete |
| `.claude/rules/` | Behavioural guardrails, auto-loaded each session |
| `templates/` | Post folder template (`post.yaml`, `notes.md`, `placeholder.md`) |
| `posts/` | Per-post working folders with artefacts |
| `posts/INDEX.md` | TOC only — read this before brainstorming or ideating to see all covered topics at a glance (cheap, ~50 lines) |
| `posts/index/` | Per-topic card files — read the relevant `<topic>.md` for detailed summaries and paths; do NOT crawl post folders directly |

---

## Available Skills

| Skill | Description |
|-------|-------------|
| `/import-pdf` | Convert a PDF reference post to clean markdown |
| `/new-post` | Full pipeline orchestrator — new post or `--from-draft` mode |
| `/brainstorm` | Interactive brainstorm → `post.yaml` + expanded notes |
| `/research` | Validate + enrich URLs, fill gaps, write `research_brief.md` |
| `/draft` | Style-grounded outline + long-form draft |
| `/seo` | SEO brief + title variants (any draft) |
| `/revise` | SEO-driven draft revision + post-revision SEO verification (any draft + brief pair) |
| `/review` | 3-critic multi-agent debate → 6-dimension rubric + panel consensus + publish readiness verdict |
| `/promote` | LinkedIn + Substack bundle (any draft) |
| `/index` | Append-only post ledger — pipeline posts + reference posts → `posts/INDEX.md` |

---

## Runtime Dependencies And Conveniences

### Portable repo dependencies

These are the only truly required dependencies for the workflow itself:

- Markdown-readable skill, agent, and guide files
- The repo directory structure and file naming conventions
- Ability to read and write files in `posts/<slug>/`
- Ability to update `post.yaml` and stage artefacts predictably

### Provider-specific conveniences

These are helpful in Claude Code but are not required for another agent to operate the repo:

| Tool | Purpose |
|------|---------|
| `context-mode` | Context window management — `ctx_fetch_and_index`, `ctx_execute`, `ctx_search` |
| Chrome DevTools MCP | Browser automation for research gap-filling (DuckDuckGo searches via `new_page`, `fill`, `press_key`) |
| `rtk` | Token-optimised Bash proxy — rewrites all Bash commands transparently via `BASH_ENV` hook |

### Fallback expectation for other agents

If these conveniences are unavailable, continue by reading the repo files directly, performing the skill steps manually, and only skipping capabilities that genuinely require an unavailable external tool.

---

## Active Hooks

Two hooks run automatically whenever Claude Code is working in this repo. Both are registered in `.claude/settings.local.json`.

### How they chain

1. **detect-skill-complete.js** (PostToolUse/Write) — fires on every Write call. Checks whether the written file is a known skill output. If so, writes `{"skill": "...", "postFolder": "..."}` to `/tmp/.newsletter_skill_ran`.
2. **skill-reflector.js** (Stop) — fires when Claude finishes a turn. If the marker exists, reads it, deletes it, then blocks the session end and injects a reflection prompt — prompting Claude to append a reflection entry to `<postFolder>/skill_reflection_log.md`.

The marker is deleted **before** returning the block decision to prevent re-trigger on the subsequent Stop call. `stop_hook_active` is checked at entry to prevent infinite loops.

### Signal detection

Most skills are detected by the unique basename of their primary output file — see the `SKILL_SIGNALS` map in `.claude/hooks/detect-skill-complete.js` for the full list. Two exceptions:

- **`/brainstorm`** — detected by a content check on `post.yaml` (presence of `stages.brainstorm.status: complete`), because `post.yaml` is written by every skill.
- **`/import-pdf`** — detected by path prefix (`reference_posts/`), because the output filename is slug-derived at runtime.
- **`/new-post`** — not detected; it is an orchestrator with no unique output file of its own.

### Adding a new skill signal

Add an entry to the `SKILL_SIGNALS` map in `.claude/hooks/detect-skill-complete.js`. If the output filename is not unique, add a content or path-prefix fallback after the map lookup — see the `brainstorm` and `import-pdf` blocks as examples.

---

## Rules

Behavioural and maintenance rules live in `.claude/rules/` and are loaded automatically:

- `.claude/rules/core-rules.md` — content, workflow, and writing guardrails
- `.claude/rules/maintenance-rules.md` — what to update when significant changes happen
- `.claude/rules/output-limits.md` — guardrail against losing long-form writes to the per-response output token cap

---
> Source: [JoseParrenoGarcia/newsletter-engine](https://github.com/JoseParrenoGarcia/newsletter-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
