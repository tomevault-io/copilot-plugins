## agentic-skills

> This repository defines a universal agentic development pack. It works for any project type: web, mobile (iOS/Android), desktop, server, game (Godot/Unreal/Unity), AI/LLM agent, CLI, or library. It gives Codex, Claude Code, Cursor, Aider, and other MCP-compatible agents permanent project rules, reusable skills, and an explicit phase graph.

# Agentic Skills — Universal AI-Agent Development Rules

## Purpose

This repository defines a universal agentic development pack. It works for any project type: web, mobile (iOS/Android), desktop, server, game (Godot/Unreal/Unity), AI/LLM agent, CLI, or library. It gives Codex, Claude Code, Cursor, Aider, and other MCP-compatible agents permanent project rules, reusable skills, and an explicit phase graph.

## Repo Map

- `agentic/`: main system directory for skills, routing, docs, scripts, Obsidian skeleton.
- `agentic/skills/`: 46 skill folders, each with `SKILL.md`, `agents/openai.yaml`, and `references/`.
- `agentic/routing/skills.json`: machine-readable routing by entrypoint, phase, permission, combinations.
- `agentic/routing/principal-operating-model.md`: principal-level checklist for risky work.
- `.claude/rules/`: Claude-specific rule files (architecture, testing, performance, security, coding conventions).
- `agentic/obsidian/project-skeleton/`: Markdown artifacts for project memory (intake, discovery, product, architecture, delivery, quality, security, UX/a11y/i18n, data/ML, release, compliance, platform, init, devops, existing-product, agent learning).
- `install.sh`: global or project-local skill installer.
- `DESIGN.md`: documentation design contract.
- `agentic/scripts/validate.py`: structural validation.

## Default Workflow

1. **Init first.** Use `init-project` at the start of any project; fingerprint existing repo or scaffold a fresh one.
2. **Lock platforms.** Use `platform-detector` to set the target-platform matrix and assign security/accessibility/performance/test floors.
3. **Project structure governance.** Use `project-structure-governance` to keep a clear repo map, isolate temporary/test artifacts, maintain one docs/runbook entrypoint, and remove root clutter before handoff.
4. **Hypothesis discipline.** Run every load-bearing claim through `hypothesis-validator` with a kill criterion.
5. **Phase ladder.** Use `ai-pdlc` for phase-level evidence; `sdlc-orchestrator` for operational routing.
6. **Read-only by default.** Keep intake, research, architecture, decomposition, QA, and review read-only until gates pass.
7. **TDD before code.** `tdd-workflow` writes the failing test and implementation contract; `service-implementation` writes production code against that contract; TDD then verifies green/refactor/anti-overfit evidence.
8. **Context7 by default.** Use Context7 MCP before any external-library, framework, API, CLI, platform, OWASP/MASVS, or CVE-feed format lookup. Query live CVE/KEV/GHSA sources separately and record evidence.
9. **Security ladder.** Before release, run matching `security-owasp-{web,llm,agentic}`, `security-mobile-masvs` for native iOS/Android, plus `cve-zero-day-scanner`. KEV-listed findings block release.
10. **UX, a11y, i18n.** Every user-facing surface goes through `ux-design`, `accessibility-audit`, and `i18n-localization` before release if multilingual or in a regulated accessibility market.
11. **Data & ML discipline.** Any data product or LLM/RAG feature goes through `data-ml-pipeline` for contracts, eval harness, and drift monitoring.
12. **Compliance & release.** Use `compliance-legal` to map applicable frameworks (GDPR/CCPA/HIPAA/PCI/SOC2/EU AI Act/etc.) and `release-management` to pull every gate verdict into a single go/no-go with rollback runbook.
13. **Agent harness.** Use `agent-harness-architect` before adding broad tool access, persistent memory, sub-agents, unattended loops, external execution, or agent-platform routing.
14. **Self-improvement event loop.** Use `self-improvement-loop` during an active task only from failed checks, logs, evals, user feedback, route mismatch, or regression signals. Run one bounded observe/analyze/repair/validate/memory cycle, then resume the original task.
15. **Custom skills.** Use `custom-skill-builder` for user-defined skills so frontmatter, references, routing, metadata, and validation stay consistent.
16. **Graph hygiene.** Update Obsidian project memory via `documentation-graph-curator`; subsequent reads via `obsidian-graph-navigator` to save tokens.
17. **Coding conventions.** Apply `.claude/rules/coding-conventions.md` to every line of code. Comments are written in the user's session language. No decorative ALL-CAPS, no marketing voice, no banner separators. Comments explain the non-obvious WHY only.
18. **Principal bar.** Apply `agentic/routing/principal-operating-model.md` to risky or cross-cutting work before any write-heavy step.

## Definition of Done

- Init mode (fingerprint / scaffold / hybrid) is recorded.
- Platform matrix is locked.
- Project structure contract, scratch/test artifact workspace, and canonical docs/runbook entrypoint are recorded.
- Hypotheses have kill criteria and current evidence.
- Production code shipped through TDD contract, service implementation, and anti-overfit test.
- Context7 MCP documentation validation is recorded when external libraries, APIs, CLIs, platforms, OWASP/MASVS wording, or CVE feed formats are involved; live vulnerability facts have feed/API/CLI evidence.
- Agent harness changes include contracts for routing, tools, memory, evaluation, permissions, telemetry, and handoff.
- Self-improvement changes include trigger event, repair surface, regression check, Obsidian learning note when useful, before/after result, and resume point.
- OWASP-track per platform is Pass; CVE register has no unresolved KEV-listed findings; release verdict is Go (or Conditional with named approver and expiry).
- Affected artifacts in the Obsidian graph are updated; broken wikilinks resolved.
- Tests or validation commands are run or clearly reported as not run.
- Risks, rollback, and follow-up work are documented.
- Installer changes are checked with `./install.sh --dry-run` before real installation.
- GitHub-facing docs keep the root README universal and put translations in `agentic/docs/` (RU primary).

## Forbidden Defaults

- Do not start coding in a new directory without `init-project`.
- Do not jump from a broad request directly to code.
- Do not place one-off scripts, generated outputs, screenshots, temp logs, or debug probes in the repository root.
- Do not create competing setup/runbook docs; update the canonical entrypoint instead.
- Do not treat opinions as decisions; route through `hypothesis-validator`.
- Do not write production code before a failing test exists.
- Do not rely on model memory for current API, library, framework, platform, CLI, OWASP, or CVE behavior when Context7 MCP can verify it.
- Do not release without the matching OWASP-track and CVE/zero-day scan passing.
- Do not add autonomy, persistent memory, broad tools, or multi-agent delegation without an explicit harness contract.
- Do not run self-improvement loops from vague dissatisfaction; require evidence, a regression check, and a clear return path to the user's original task.
- Do not spawn multiple agents without disjoint ownership and merge order.
- Do not treat Mermaid links as Obsidian graph links.
- Do not write machine-specific absolute paths into tracked docs.
- Do not silently overwrite existing `CLAUDE.md`/`AGENTS.md`/`.codex/`/`.claude/` in user projects.

---
> Source: [ashermahonin/agentic-skills](https://github.com/ashermahonin/agentic-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
