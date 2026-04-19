## repo-intel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

repo-intel is a prompt-driven AI repository analysis platform. It ingests software repositories (GitHub, GitLab, Azure DevOps) and generates comprehensive structured intelligence reports across six dimensions: code quality, architecture, security, DevOps, dependency risk, and governance. The system is composed of Markdown skill definitions and prompt orchestration, executed via Claude Code CLI.

## How It Works

The entry point is **ANALYZE.md**, which orchestrates the full analysis pipeline:
1. Accepts a remote repo URL or local folder path
2. Runs a discovery scan to identify tech stack, all projects/modules, and framework versions
3. Loads skill modules from `skills/*.md`
4. Executes skills using a phased approach:
   - **Phase 1:** Security first (blocking — escalates critical secrets immediately)
   - **Phase 2:** Code, Architecture, DevOps, Dependency in parallel (using Claude subagents)
   - **Phase 3:** Governance and Claude Metrics (sequential, after Phase 2)
5. Synthesizes cross-skill insights (root causes, patterns, systemic issues)
6. Generates a self-contained HTML report (`repo-intel-report.html`) using `output/report-template.html`
7. Cleans up cloned repos (remote URLs only)

Skills are pluggable — each `skills/*.md` file defines an independent analysis dimension. See `skills/HOW-TO-ADD-SKILL.md` for adding new ones.

## Analysis Scope

The platform analyzes **all projects in a solution** — for .NET solutions with multiple .csproj files, Java multi-module Maven projects, or monorepos with packages/apps. It does not limit findings to "top 3" or "top 5" — it reports everything discovered.

Language/framework coverage includes: .NET/C#, Java/Kotlin (Spring Boot), Python (Django, Flask), Node.js/TypeScript (React, Next.js, Angular), Go, Rust, Ruby/Rails, PHP/Laravel, Dart/Flutter, and more.

## CI and Linting

CI runs on GitHub Actions (`.github/workflows/ci.yml`) on every push to main and all PRs.

```bash
# Install linting tools
npm install -g markdownlint-cli2 markdown-link-check

# Lint all Markdown
markdownlint-cli2 "**/*.md" "#node_modules"

# Check links in a specific file
markdown-link-check <file.md>
```

## Key Directories

- `skills/` — Analysis skill modules (the core logic of the platform)
- `output/` — Report templates (HTML with dark/light mode, Mermaid diagrams, and Markdown)
- `docs/` — Platform design documentation (data pipeline, knowledge graph, API, scaling, roadmap)
- `architecture/` — System architecture design (7-layer model, microservices, data flow)
- `engineering/` — Coding standards, module design guidelines, plugin/skill framework
- `agents/` — AI agent specifications for each analysis domain
- `examples/` — Sample generated reports

## Architecture Principles

- AI augments static analysis — AI reasons over structured findings, not raw code
- Skill-based extensibility — new analysis dimensions don't require changes to the orchestration core
- Parallel execution — independent skills run as Claude subagents for throughput
- Skills run independently — one failure doesn't cascade to others
- Exhaustive analysis — all projects, all files, all findings; never cap output
- Ephemeral clones — no persistent storage of analyzed repository code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sumsankar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
