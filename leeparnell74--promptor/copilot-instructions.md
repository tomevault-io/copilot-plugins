## promptor

> - Mission: turn vague requests into precise, actionable prompts for AI-driven development.

# Repository Guidelines

## Core Principles
- Mission: turn vague requests into precise, actionable prompts for AI-driven development.
- No‑Fluff: concise, measurable, and practical outputs with honest feedback.
- Domains: Product, Architecture, Development, Testing/QA, and DevOps.
- Sub‑agents: role prompts under `prompts/claude-agents/` (mirrored in `.claude/agents/`).

## Release status
- Current version: v0.2.0
- Highlights:
  - Engine: Implemented CLI with extended presets (tone inference, candidates/validate/score/redact)
  - Web UI: Single-screen FastAPI app with live format toggle and copy actions
  - VS Code: Enhance selection, Generate From Pack, Fill From JSON; host/port configuration
  - Presets/Patterns: ~25 patterns; added Architect/Analyst/Marketing patterns; packs for PM/Dev/QA/DevOps
  - Demos/Automation: `make demo`, `make lint`, `scripts/demo.sh`
- Next up:
  - Populate `test-results.md` with real runs
  - Expand examples/templates as needed

## Project Structure & Module Organization
- Root docs: `README.md`, `CLAUDE.md`, `testing-guide.md`, `test-results.md`.
- Prompts: `prompts/<domain>/...` (e.g., `architecture-design/`, `prompt-enhancement/`, `testing-qa/`, `claude-agents/`).
- Claude agent mirrors: `.claude/agents/*.md` should reflect `prompts/claude-agents/*.md` one-to-one.
- Best practices: `best-practices/` (integration notes, patterns).
- Templates: `templates/` for reusable prompt skeletons.
- Editor config: `.vscode/settings.json` for recommended local settings.

## Build, Test, and Development Commands
- No build step; content is Markdown-only.
- Search repository: `rg -n "TODO|TBD|FIXME" prompts best-practices` (quick hygiene pass).
- List prompts: `rg --files prompts | sort` (overview of content).
- Preview Markdown locally using your editor’s Markdown preview (e.g., VS Code).

## Coding Style & Naming Conventions
- Files: kebab-case Markdown, e.g., `prompt-enhancer-agent.md`.
- Headings: ATX style (`#`, `##`), sentence case, concise.
- Formatting: wrap lines ~80–100 chars, use fenced code blocks for commands (`bash`).
- Links: use relative links where possible; wrap paths and commands in backticks.
- Agents: keep filenames and titles synchronized between `prompts/claude-agents/` and `.claude/agents/`.

## Pattern Pillars (from CLAUDE.md)
- Structure (Repo Prompt): role, numbered tasks, and “for each” loops.
- Precision (No‑Fluff): metrics and checkable criteria (e.g., p95 < 100ms).
- Orchestration (MCP): tool usage, error recovery, and resource limits.
When adding patterns, keep constraints measurable and deliverables verifiable.

## Testing Guidelines
- Follow `testing-guide.md` for process, and record notable outcomes in `test-results.md`.
- Validate links and anchors in Markdown preview; fix broken references before submitting.
- Sanity-check examples: copy/paste commands and confirm they run or are clearly illustrative.

## Commit & Pull Request Guidelines
- Commits: imperative, concise subject (≤72 chars). Example: `Fix ASCII art: complete PROMPTOR logo` or `Add repo prompt pattern`.
- Scope hint (optional): prefix with area, e.g., `prompts: add QA checklist`.
- PRs: include a clear summary, list changed files/sections, link related issues or docs, and note any follow-ups.
- Keep diffs focused; update mirrors (e.g., `.claude/agents/` ↔ `prompts/claude-agents/`) in the same PR.

## Agent-Specific Instructions
- New agent prompt: add to `prompts/claude-agents/<agent-name>.md` and mirror at `.claude/agents/<agent-name>.md` with matching title and content.
- Include: purpose, inputs, usage steps, and example prompts. Keep examples minimal and copy-ready.
- Avoid secrets or credentials in any examples; redact or use placeholders.

## Architecture Overview
- Current: Implemented documentation-first repository organizing prompt assets by domain and role, plus a working enhancement engine and web UI.
- Implemented: A lightweight prompt enhancement engine that reads patterns from `engine/patterns/`, applies transformations to user inputs, and outputs enhanced prompts. Engine code lives under `engine/` with a CLI.

## Screenshots & Assets
- Store illustrative images (if any) under `assets/` with kebab-case names; reference using relative paths (e.g., `![flow](assets/prompt-flow.png)`). Keep assets under 200KB where possible.

## Engine Usage (Implemented)
- Enhance (non-interactive): `python3 -m engine.cli enhance -i "vague request" --non-interactive`
- Enhance (interactive Q&A): `python3 -m engine.cli enhance -i "vague request" --ask 3`
- Presets: `--preset bug-fix|code-review|prd|bug-report|test-plan|outline|api-design|arch-system|arch-tradeoff|ba-brd|ba-process|mkt-brief|mkt-messaging|deploy-plan|runbook|postmortem`
- Formats: `--format markdown|chatgpt|claude|gemini|chat`
- Candidates: `--candidates 2` to see 2 alternatives (max 3)
- Validate: `--validate` to print structure/length warnings before output
- Score: `--score` to print a simple 0–5 quality score
- Redact: `--redact` to remove sensitive information
- Output: copy-ready prompt(s) suitable for ChatGPT/Claude/Gemini
  - Optional: copy `.claude/agents/*.md` to `~/.claude/agents/` for global agent access in Claude Code.

## Patterns
- Location: `engine/patterns/*.json` (e.g., `code_fix.json`, `product_prd.json`, `architect_tradeoff_analysis.json`).
- Fields: `category`, `tags`, `role`, `deliverables[]`, `constraints[]`, `id`.
- Selection: engine scores tag matches and prefers the detected category; presets map to categories.
 - Included presets map to patterns: `bug-fix`→code, `prd`→product, `bug-report`→qa, `api-design`→code, `arch-system`→architecture, etc.

## Sessions (Web API)
- Create: `POST /session` with `{ input, goal?, audience?, tone?, ask?, preset? }` → returns `id` and `questions[]`.
- Answer: `POST /session/{id}/answer` with `['answer1', 'answer2', ...]` → returns `prompt`.
- Direct: `POST /enhance` supports one-shot generation (with or without questions, accepts `preset` and `format`).

---
> Source: [leeparnell74/promptor](https://github.com/leeparnell74/promptor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
