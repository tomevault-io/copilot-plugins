## research-skills

> This repo is a **skills collection for OpenCode/Hermes** — not a code project with tests or builds. The skills live under `skills/` and are loaded on demand via the `skill` tool.

# Research-Skills AGENTS.md

This repo is a **skills collection for OpenCode/Hermes** — not a code project with tests or builds. The skills live under `skills/` and are loaded on demand via the `skill` tool.

## First Things

**Read `skills/for_system_prompt.md`** — it's the meta-system prompt for the Academic Research Copilot role. Inject its content into your system prompt before using `paper-writing` or `patent-writing`.

## Primary Entry Points

Two umbrella skills cover the full academic workflow:

- **`skills/paper-writing/SKILL.md`** — 7-step paper writing pipeline (intent→feasibility→deep research→drafting→consistency→plagiarism→humanize)
- **`skills/patent-writing/SKILL.md`** — 7-step patent drafting pipeline (same structure)

Both reference sub-modules in `skills/paper-writing/references/`.

## Important Gotchas

### DO use paper-writing SKILL.md directly, NOT research-* sub-skills
The README documents `research-intent`, `research-idea-parser`, etc. as separate directories — **they don't exist as separate dirs anymore**. The full pipeline logic is embedded directly in `paper-writing/SKILL.md` and `patent-writing/SKILL.md`. Route via `for_system_prompt.md` instead.

### docx generation pitfalls (from paper-writing SKILL.md)
- For long Chinese documents (>20,000 characters), **use python-docx** not docx-js
- docx-js has f-string escaping issues with Chinese Unicode (`\u201c`/`\u201d` conflict with JS string delimiters)
- Subagent timeout: **do NOT delegate final docx generation to subagents** — they will time out (600s+). Generate in the main session.

### Data currency (paper-writing SKILL.md)
- Distinguish **Current Status** (past real data) from **Trends/Forecast** (future-only)
- Always ask user for "current time point" before writing
- Label data sources with cutoff dates
- SIA annual report, FAA, planet4589.org, UCS satellite database are top data sources

### Bibliometric papers
If keywords include "bibliometric", "文献计量", "VOSviewer", etc., **do NOT run the standard 7-step workflow**. Read `references/bibliometric-paper-workflow.md` first — the flow is fundamentally different (real data collection via CrossRef API, analysis + visualization, data export as CSV).

### arXiv skill
Uses curl + arXiv REST API (no API key). Parse Atom XML output with inline Python.

### Language
The repo is bilingual (Chinese/English). Primary documentation is in Chinese. Questions may come in either language.

## Available Skills Under skills/

| Directory | Description |
|-----------|-------------|
| `arxiv/` | arXiv paper search (curl + REST API) |
| `blogwatcher/` | RSS/Atom feed monitoring |
| `docx/` | Word document creation/editing |
| `paper-writing/` | Umbrella: academic paper writing (main) |
| `patent-writing/` | Umbrella: patent drafting |
| `opencode-integration/` | Use OpenCode as subagent via ACP |
| `fireworks-tech-graph/` | Technical diagrams as SVG+PNG |
| `software-development/markitdown-converter/` | Doc→Markdown conversion |
| (18 DESCRIPTION.md-only) | Categorized domains: research, apple, creative, data-science, diagramming, domain, email, gaming, gifs, github, inference-sh, mcp, media, mlops, note-taking, productivity, smart-home, social-media |

## Tools Required

- `curl` — arXiv API
- `blogwatcher-cli` — RSS monitoring
- `python-docx` / `docx-js` — .docx generation
- `pandoc` — doc conversion
- `matplotlib` — chart generation (install via pip with `--break-system-packages`)
- `cairosvg` — SVG→PNG conversion (fireworks-tech-graph)
- `markitdown` — doc→markdown conversion (installed at `~/.local/bin/markitdown`)

## Licensing

MIT License © 2026 nors1993

---
> Source: [nors1993/Research-Skills](https://github.com/nors1993/Research-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
