## ppt-agent

> Always answer in Chinese (Simplified).

# PPT Agent Plugin

Always answer in Chinese (Simplified).

<available-skills>

| Skill             | Trigger                                                                   | Description                                                              |
| ----------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `/ppt-agent:ppt`  | "PPT", "slides", "幻灯片", "做个PPT", "slide deck", "演示文稿", "做演示" | PPT slide generation workflow: init → research → outline → design → delivery |
| `agent-reach`     | "search Twitter/Reddit/YouTube", "read this URL", "搜索", "查一下"        | Self-contained multi-platform search & read (zero external dependencies) |

</available-skills>

## Available Commands

- `/ppt-agent:ppt`: End-to-end PPT slide generation workflow.

## Workflow Phases

1. Init: Parse arguments, create `openspec/changes/<run_id>/`.
2. Requirement Research (Hard Stop): Background search + user clarification.
3. Material Collection (Parallel): Per-section deep search.
4. Outline Planning (Hard Stop): Pyramid Principle outline + user approval.
5. Planning Draft: Simple SVG pages per slide (per-slide signaling enables Phase 6 pipeline).
6. Design Draft + Gemini Review: Bento Grid SVG generation + quality review loop.
7. Delivery (Hard Stop): Collect final SVGs + HTML preview + summary.

## Agent Types

- `ppt-agent:research-core` — requirement research + material collection
- `ppt-agent:content-core` — outline planning + planning draft
- `ppt-agent:slide-core` — design SVG generation with Bento Grid
- `ppt-agent:review-core` — Gemini-powered SVG layout & aesthetic optimization

All agents are spawned via `Task()` calls. No Agent Team required — agents communicate only with the lead orchestrator via `SendMessage` (heartbeat, ready, error signals), not with each other directly.

## Skill Invocation Conventions

- `ppt-agent:research-core` uses `agent-reach` skill for web search (self-contained, no external installer needed — probes and uses whatever upstream tools are available on the machine).
- `ppt-agent:review-core` invokes `ppt-agent:gemini-cli` for SVG layout & aesthetic optimization. Gemini's raw outputs (`gemini-raw-*.md`) are preserved as intermediate artifacts in `${RUN_DIR}/reviews/`.
- Prompt references are in `skills/_shared/references/prompts/`.
- Style tokens are in `skills/_shared/references/styles/`. Available styles are discovered from `skills/_shared/index.json` (domain=style).
- HTML preview template is in `skills/_shared/assets/preview-template.html`.
- `outline.json` schema is defined in `skills/_shared/references/prompts/outline-architect.md` (single source of truth).
- Optimization fallback policy is defined in `skills/gemini-cli/SKILL.md` (single source of truth). When Gemini is unavailable, the workflow degrades to technical validation only — hard-rule checks continue, but aesthetic optimization is not faked.
- Brand color override is supported via `--brand-colors=<path>` flag. Brand styles are written to `${RUN_DIR}/brand-style.yaml`.

## HTML Preview Template

The delivery phase generates an interactive `index.html` alongside the SVG files. The template uses 4 placeholders:

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{{TITLE}}` | `outline.json` title | `新一代小米SU7发布会` |
| `{{LOGO}}` | Short brand mark (2-3 chars) | `Mi`, `PPT` |
| `{{ACCENT_COLOR}}` | Style YAML accent hex | `#FF6900` |
| `{{SLIDES_JSON}}` | JSON array from outline.json | `[{"file":"slide-01.svg","label":"封面"}]` |

Preview modes: Gallery (thumbnail grid) / Scroll (vertical) / Present (fullscreen).
Navigation: click left/right half of screen or `←→` keys to navigate slides.
Keyboard: `P` present / `G` gallery / `S` scroll / `F` fullscreen / `N` notes / `Esc` exit.

## Quality Gates

When Gemini review is available:

- Layout score >= 7/10
- No critical review issues
- Maximum 2 fix rounds per slide
- All slides must be valid SVG 1280x720

When Gemini is unavailable:

- Fall back to technical validation only
- Enforce XML validity, `viewBox`, font-size floor, safe area, contrast, and style-token compliance
- Skip aesthetic scoring and fix suggestions

## Output Directory

- `research-context.md`
- `requirements.md`
- `materials.md`
- `outline.json` + `outline-preview.md`
- `drafts/slide-{nn}.svg`
- `draft-manifest.json`
- `slides/slide-{nn}.svg`
- `slide-status.json` — per-slide incremental progress (enables `--run-id` resume)
- `reviews/review-{nn}.md`
- `reviews/review-holistic.md`
- `review-manifest.json` — Phase 6→7 quality gate checkpoint
- `output/` — final deliverables:
  - `slide-{nn}.svg` — final design SVGs
  - `index.html` — interactive HTML preview page
  - `speaker-notes.md` — extracted presenter notes and timing guide

---
> Source: [zengwenliang416/ppt-agent](https://github.com/zengwenliang416/ppt-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
