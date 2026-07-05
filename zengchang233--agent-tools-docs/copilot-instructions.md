## agent-tools-docs

> This repository is a curated documentation workspace for frontier AI tooling. Contributors and coding agents should treat it as a public knowledge base, not as a scratchpad.

# AGENTS.md

This repository is a curated documentation workspace for frontier AI tooling. Contributors and coding agents should treat it as a public knowledge base, not as a scratchpad.

## Repository role

`agent-tools-docs` turns interesting GitHub repositories into practical, structured notes for developers who use Claude Code, Codex, MCP tools, AI gateways, and related agent ecosystems.

The expected output is not a short bookmark entry and not a direct README translation. Each document should help a reader answer:

- What is this project for?
- How does it work internally?
- How do I install and use it?
- How can it fit into Claude Code / Codex workflows?
- When should I avoid it?
- What files or docs should I read next?

## Public vs local files

Keep the public repository clean.

Tracked/public files should generally be:

```text
README.md
README.zh-CN.md
AGENTS.md
_templates/
coding-agents/
ai-gateways/
<future-category>/
```

Do **not** commit local planning memory or scratch logs:

```text
task_plan.md
findings.md
progress.md
.planning/
.DS_Store
```

Local planning files may exist as ignored files or symlinks for `planning-with-files`, but they must not be committed.

## Language policy

- The default repository README is English: `README.md`.
- The Chinese README is `README.zh-CN.md` and should be linked near the top of `README.md`.
- Long-form project notes are currently written primarily in Simplified Chinese.
- Keep technical product names, CLI flags, package names, file paths, APIs, and protocol names in their original English.
- Use clear Chinese explanations for concepts and workflow guidance.
- Avoid machine-translation-style wording; write like a practical engineering handbook.

## Writing style

Write in a concise but useful documentation style:

- Prefer structured sections, bullet lists, and tables.
- Start with an actionable summary before implementation details.
- Be specific about commands, config files, entry points, and risks.
- Distinguish verified facts from interpretation or recommendation.
- Avoid hype, vague praise, and unsupported claims.
- Explain both strengths and limitations.
- Include practical Claude Code / Codex usage patterns, not just generic installation steps.
- When a project has safety, permission, billing, account, browser, shell, or network risks, call them out explicitly.

Tone target:

- practical;
- technically precise;
- neutral and comparative;
- optimized for developers who may actually try the tool.

## Document template

For a new repository note, use `_templates/repo-analysis-template.md` unless there is a strong reason to adapt it.

The standard section order is:

1. 一句话总结
2. 项目目的与核心问题
3. 核心能力
4. 实现思路 / 架构拆解
5. 安装与快速开始
6. 在 Claude Code 中的用法
7. 在 Codex 中的用法
8. 典型生产力场景
9. 适合 / 不适合
10. 同类工具定位
11. 关键文件/目录导读
12. 学习与掌控建议
13. Sources

Do not omit the Claude Code and Codex sections unless the project is clearly unrelated to agent-assisted development.


## Default behavior for repo-summary requests

When a user asks to summarize, analyze, or explain the usage of a GitHub repository while working in this workspace, treat the default deliverable as a public repository note, not just a chat-only answer, unless the user explicitly asks for a conversational summary only. The default workflow is:

1. Verify the official repository and current primary docs.
2. Create or update `<category>/<repo-name>.md` using the standard section order.
3. Update the matching index tables in both `README.md` and `README.zh-CN.md`.
4. Keep planning files and scratch research local and unstaged.
5. If the user asks to push or says to continue until push, run the commit hygiene checks, commit only intended public files, push to the configured remote, and verify the final working tree.

## Research workflow for new repos

When adding or updating a project note:

1. Verify the official repository URL and current docs.
2. Prefer primary sources: GitHub README, official docs, package metadata, source files, release notes, and configuration schemas.
3. Inspect source structure when the README is not enough.
4. Identify the correct category directory before writing.
5. Create or update `<category>/<repo-name>.md`.
6. Add the document to the appropriate table in both `README.md` and `README.zh-CN.md`.
7. Keep URLs and source references in the `Sources` section of the project note.

If information can change quickly, include a `Last checked: YYYY-MM-DD` line and, where useful, a checked commit/version.

## Category conventions

Use these directories unless a new category is clearly needed:

| Category | Use for |
|---|---|
| `coding-agents/` | Terminal, IDE, or plugin-based AI coding agents, workflows, and review tools |
| `agent-frameworks/` | Frameworks for agent apps, multi-agent systems, and workflow runtimes |
| `mcp-tools/` | MCP servers, clients, and ecosystem tools |
| `llm-app-frameworks/` | LLM app frameworks, RAG, prompt tooling, and workflow orchestration |
| `ai-gateways/` | API gateways, relays, account pools, billing, and model routing |
| `browser-automation/` | Browser control, web automation, visual agents, and interaction tooling |
| `eval-observability/` | Evaluation, tracing, replay, observability, and quality monitoring |
| `devtools/` | General developer productivity tools |
| `data-tools/` | Data processing, retrieval, indexing, crawling, and knowledge-base tools |
| `research-projects/` | Papers, experiments, and research prototypes |
| `uncategorized/` | Temporary location only when classification is unclear |

If adding a new category, update both READMEs and keep the naming lowercase kebab-case.

## README maintenance

When adding a new public note:

- Update the documentation index in `README.md`.
- Update the matching index in `README.zh-CN.md`.
- Keep the English README as the default entry point.
- Keep badges at the top unless they break rendering.
- Keep the Chinese README link near the top of the English README.

## Source and citation expectations inside notes

Project notes should end with a `Sources` section containing official or primary links.

Prefer sources such as:

- official GitHub repository;
- official docs;
- package registry page;
- relevant source files;
- release notes or changelog;
- official examples.

Avoid relying on blog posts, social media, or third-party summaries unless they are clearly marked as secondary context.

## Safety and accuracy

- Do not invent support for tools, providers, protocols, or models.
- If a feature is inferred from source code, say so or point to the relevant file.
- For tools that execute shell commands, edit files, use browsers, connect to accounts, or proxy API traffic, include risk and permission guidance.
- For API gateways, account pools, billing, or subscription-related projects, mention compliance and upstream Terms-of-Service risk when relevant.
- For rapidly evolving projects, prefer wording like “current docs describe…” rather than timeless claims.

## Commit hygiene

Before committing:

```bash
git status --short
git diff --check
```

Make sure ignored local planning files are not staged. Commit messages should be short and descriptive, for example:

```text
Add oh-my-pi coding agent notes
Add bilingual README
Hide local planning files
```

## Agent behavior in this repository

When acting as an AI coding agent here:

- Be careful not to expose local planning memory.
- Preserve the established documentation structure and tone.
- Prefer improving reader-facing docs over expanding private logs.
- Keep changes small and reviewable unless the user explicitly asks for a large reorganization.
- If asked to research a new repo, use current official sources and record the check date in the note.
- If asked to push, commit only the intended public files and verify the working tree afterward.

---
> Source: [zengchang233/agent-tools-docs](https://github.com/zengchang233/agent-tools-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
