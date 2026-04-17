## ashford

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ashford** is a Claude Code plugin marketplace that provides slash commands, agents, skills, hooks, and MCP server configurations. It's installed via the Claude Code marketplace system (`/plugin marketplace add` + `/plugin install`) and all components are auto-discovered at runtime.

- **Author:** Luqman Nurhakim
- **No build system** — all components are Markdown/JSON files auto-discovered by Claude Code. No compilation, bundling, or package management needed for the plugin itself.

## Architecture

The marketplace contains three plugins: `egg/` (development workflows), `aerion/` (job application tracking), and `dunk/` (DLN learning system). Claude Code discovers all components by convention:

| Component | Location | Format |
|-----------|----------|--------|
| Commands | `egg/commands/*.md` | Markdown with YAML frontmatter |
| Agents | `egg/agents/*.md` | Markdown with YAML frontmatter |
| Skills | `egg/skills/*/SKILL.md` | Markdown with YAML frontmatter |
| MCP Servers | `egg/.mcp.json` | JSON config |
| Hooks | `egg/hooks/hooks.json` | JSON config |
| Helper Scripts | `egg/scripts/` | Shell scripts |

The top-level `.claude-plugin/marketplace.json` is the marketplace registry pointing to the plugin directory. The `templates/` directory has ready-to-copy `.mcp.json` configurations (personal, all).

## Current Components

- **`/commit`** — Conventional Commits 1.0.0 compliant commit creation with diff analysis. Explicitly forbids AI attribution footers.
- **`/status`** — Project status overview (git state, recent changes, pending tasks).
- **`/debug-ccskill`** — Skill bug diagnosis via execution path tracing, failure pattern matching, and root cause classification.
- **`code-reviewer` agent** — Code review for quality, security, and performance.
- **`leetcode-profile-sync` agent** — Internal agent for leetcode-teacher profile/ledger I/O. Dispatched at skill activation, resumed at write-back. Not user-facing.
- **`mlx-dev` skill** — Apple MLX development guide with references in `skills/mlx-dev/references/`.
- **`doc-generator` skill** — Automated documentation generation.
- **`ml-paper-writing` skill** — ML research paper writing assistance.
- **`tech-blog` skill** — Technical blog post generation for Jekyll with KaTeX math and BibTeX citations.
- **`resume-analyzer` skill** — JD analysis, keyword extraction, gap analysis, and mitigation strategies. Outputs `notes.md` for `resume-tailor`.
- **`resume-tailor` skill** — Generates tailored LaTeX resume from `resume-analyzer` notes using XYZ bullets and ATS optimization.
- **`cover-letter` skill** — Standalone cover letter generation from analysis notes.
- **`leetcode-teacher` skill** — Socratic LeetCode/ML implementation teacher with evidence-based learning science. 50 reference files covering algorithm frameworks, problem patterns, data structures, sorting, linked lists, arrays, binary search, stacks/queues, matrix techniques, graphs, brute-force search, dynamic programming, greedy algorithms, combinatorics, numerical methods, advanced graph algorithms, linear algebra, and ML implementations.
- **`global-markets-teacher` skill** — Socratic global markets & trading interview teacher with 3 modes (Learning, Recall, Mock Interview). 28 reference files covering rates/fixed income, FX, equities, commodities, credit, crypto, macro economics, derivatives, risk management, market mechanics, trade pitch framework, scenario analysis, headline analysis, brain teasers, and 6 firm-type profiles (hedge funds, banks, asset managers, trading houses, energy majors, crypto trading firms).
### aerion plugin (`aerion/`)

- **`/check-apps`** — Scans Gmail for job application status updates and syncs to a Google Sheets "Job Tracker" in the hojicha Drive folder. Uses workspace MCP (Gmail) + mcp-google-sheets (Sheets read/write).
- **`job-tracker` skill** — Email classification rules, entity extraction, stage progression logic, and sheet update constraints for job application tracking.

### dunk plugin (`dunk/`)

- **`dln` skill** (orchestrator) — Domain-agnostic learning orchestrator using the Dot-Linear-Network cognitive topology framework. Parses domain, queries/creates Notion profile, routes to appropriate phase skill.
- **`dln-dot` skill** — Dot phase (70% delivery / 30% elicitation). Builds foundational concept nodes and causal chains for new domains.
- **`dln-linear` skill** — Linear phase (50/50 delivery/elicitation). Discovers shared factors across procedural chains, builds transferable understanding.
- **`dln-network` skill** — Network phase (20% delivery / 80% elicitation). Stress-tests, compresses, and refines the learner's mental model via the Distributed Revision Cycle.
- **`dln-compress` skill** — Internal compression format for dln-sync agent. Not user-facing.
- **`dln-sync` agent** — Internal Notion I/O agent for DLN running ledger. Dispatched by phase skills at teaching boundaries.
- **`dln-syllabus` agent** — Internal curriculum researcher. Dispatched by DLN orchestrator for new domains.

## Python Linting Hook

A `PostToolUse` hook (`hooks/hooks.json`) runs `scripts/python-lint.sh` on every Write/Edit of `.py` files. It uses `ruff` with these settings:

- **Rules:** E, W, F, D, N, I (pycodestyle, pyflakes, pydocstyle, pep8-naming, isort)
- **Ignored:** D203, D213 (to enforce Google-style docstrings — D211 + D212)
- **Line length:** 100 characters
- **Style:** Google Python Style Guide + PEP 8
- **Behavior:** Auto-formats and auto-fixes first, then reports unfixable issues (exit 1). Skips silently if `ruff` is not installed.
- **Optional:** `ruff` (`pip install ruff` or `uv pip install ruff`) — hook is a no-op without it

## Frontmatter Conventions

**Commands** use: `description`, `allowed-tools`, `argument-hint`, `model` (sonnet/opus/haiku).

**Agents** use: `name`, `description` (with `<example>` blocks for trigger conditions), `model` (inherit/sonnet/opus/haiku), `color`, `tools` (array).

**Skills** use: `name`, `description` (Claude uses this to decide contextual activation).

## Adding New Components

All components are auto-discovered — create files in the right directory and they work immediately. Use `$ARGUMENTS`, `$1`, `$2` for command arguments, `@path/to/file` for file inclusion, and `!`command`` for inline bash in commands. Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths in hooks and MCP configs. Use `${ENV_VAR}` for environment variable injection in MCP server definitions.

## MCP Servers

Defined in `egg/.mcp.json`: context7 (npx), git (uvx), chrome-devtools (npx), exa (bash wrapper → hosted endpoint via mcp-remote). The exa wrapper script (`scripts/exa-mcp.sh`) expands `EXA_API_KEY` at runtime since the plugin system doesn't interpolate `${ENV_VAR}` in `.mcp.json` args. Two templates in `templates/`: `mcp-personal.json` (everything except gitlab) and `mcp-all.json` (everything including gitlab). Templates use direct mcp-remote URLs. Projects copy a template to their root `.mcp.json` and customize.

## Installation Model

**Primary (Marketplace):** Users run `/plugin marketplace add LuqDaMan/ashford` then `/plugin install egg@ashford`. This delivers all components — commands, agents, skills, hooks, and MCP servers.

**Legacy (Submodule):** Adding the repo as a git submodule at `.claude-plugins/ashford` only delivers MCP servers. Commands, agents, skills, and hooks are **not** discovered via submodules. Use the marketplace approach for full functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luqmannurhakimbazman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
