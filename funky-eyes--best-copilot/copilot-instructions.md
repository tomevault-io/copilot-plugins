## best-copilot

> This file is the local repository entry for this checkout. The plugin package supports Codex, Copilot CLI, and Claude Code: Codex install metadata lives in `.codex-plugin/` and `.agents/`, Codex custom agent adapters live in `.codex/agents/`, Copilot installable agents live in root `agents/`, Claude Code agent adapters live in root `claude-agents/`, shared installable skills live in root `skills/`, role-specific workflow skills live in `skills/*-workflow/`, and repository instructions live in `.github/instructions/**`. Platform, system, and explicit user instructions have higher priority than repository files; when there is no conflict, read the entries below.

# Repository Entry

This file is the local repository entry for this checkout. The plugin package supports Codex, Copilot CLI, and Claude Code: Codex install metadata lives in `.codex-plugin/` and `.agents/`, Codex custom agent adapters live in `.codex/agents/`, Copilot installable agents live in root `agents/`, Claude Code agent adapters live in root `claude-agents/`, shared installable skills live in root `skills/`, role-specific workflow skills live in `skills/*-workflow/`, and repository instructions live in `.github/instructions/**`. Platform, system, and explicit user instructions have higher priority than repository files; when there is no conflict, read the entries below.
This project provides plugins for Codex, Claude Code, and GitHub Copilot. As such, the .github, .agents, .codex-plugin, .claude, and .claude-plugin directories, along with AGENTS.md and CLAUDE.md, are used by this project itself rather than by the plugins it generates or manages. Unless explicitly required, do not modify the contents of these directories or files.

## Required Entries

1. `.github/instructions/must.instructions.md`: core rules, memory layers, prompt assembly, state contracts, language policy, and verification gates.
2. `.github/instructions/skills-index.instructions.md`: lightweight repo skill index.
3. `.github/instructions/project.instructions.md`: repository facts, build/test commands, and high-frequency conventions.
4. `skills/target-*-bootstrap/SKILL.md`: installable templates used to create target-local instructions, memory, and spec scaffolds after repo init.
5. `.codex-plugin/plugin.json`, `.agents/plugins/marketplace.json`, `.agents/skills`, and `.codex/agents/*.toml`: Codex plugin manifest, local/repo marketplace metadata, repo-scoped skills, and Codex custom agent adapters.
6. `claude-plugin/.claude-plugin/plugin.json` and `claude-agents/*.md`: Claude Code plugin manifest and lowercase-hyphen subagent adapters.

## Local Conventions

- Before non-trivial repository tasks, read the relevant parts of `.github/instructions/project.instructions.md` and `.github/instructions/must.instructions.md`.
- For repository initialization checks, use `repo-init-gate` first. It should read only the target root `best-copilot.md` and skip `repo-init-scan` only when the sentinel frontmatter version matches the current contract.
- When choosing a skill, read `.github/instructions/skills-index.instructions.md` first, then open only the selected skill under root `skills/`. Do not bulk-read `skills/`.
- When working in a target repository that has its own memory, progressively read that target repository's `memories/repo/INDEX.md -> current-workstreams.md -> linked_spec/linked_memory`.
- This plugin checkout does not keep active target-project `memories/**` or `spec/**`; installed projects should be initialized through the bootstrap skills.
- Edit Codex install metadata through `.codex-plugin/` and `.agents/`; edit Codex custom agent adapters through `.codex/agents/`; edit Copilot installable agents through root `agents/`; edit Claude Code adapters through root `claude-agents/`; edit shared installable skills through root `skills/`; edit repository instructions through `.github/instructions/**`.
- Do not commit or run destructive commands unless explicitly requested.
- Detect the user's primary language and answer in that language unless the user asks otherwise.

## Runtime Adapter

| Copilot Capability | Local Mapping | Boundary |
| --- | --- | --- |
| agent / handoff | `spawn_agent` + `wait_agent` / `send_input` | Use only when the user explicitly asks for agents, delegation, or parallel agent work. |
| read / search | `rg`, `sed`, `find`, `git diff`, and other read-only commands | Prefer explicit user paths and index hits; avoid unbounded scanning. |
| edit | Top-level session uses `apply_patch` | `.github/instructions/**`, `.codex/agents/**`, root `agents/`, root `skills/`, `AGENTS.md`, and bootstrap skill template changes are handled inline by the top-level session. |
| execute | `exec_command` | Real command evidence must include command and result. |
| todo | `update_plan` | Session plan only; does not replace formal spec/task state. |
| `vscode_askQuestions` / `vscode/askQuestions` / `askQuestions` / `Asking user` | Use the available native ask mechanism when present | Top-level session and PM/coordinator only. In VS Code, if `vscode_askQuestions` appears in the latest tool inventory, call that exact tool before abstract `vscode/askQuestions` / `askQuestions`; in Copilot CLI, use `Asking user` when available. Specialists must not invoke native ask and should return `NEEDS_USER_INPUT` to PM when present or `BLOCKED missing_top_level_question` otherwise. Plain prose questions cannot replace native confirmation gates. If native ask is unavailable, continue only with a single safe interpretation or report a blocked/partial state. |

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **best-copilot** (1294 symbols, 1301 relationships, 0 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/best-copilot/context` | Codebase overview, check index freshness |
| `gitnexus://repo/best-copilot/clusters` | All functional areas |
| `gitnexus://repo/best-copilot/processes` | All execution flows |
| `gitnexus://repo/best-copilot/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Source: [funky-eyes/best-copilot](https://github.com/funky-eyes/best-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
