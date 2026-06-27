## esplotter

> This repository uses `docs/**` as the guidance source of truth.

# LLM Assistant Guide

This repository uses `docs/**` as the guidance source of truth.

Before changing code or repository assets, load guidance in this order:

1. Use the `docs/` folder as the policy source of truth for task rules.
2. Before implementing changes, scan `docs/**` to find relevant documentation quickly using frontmatter `description`, then read the most relevant files fully.
3. Use execution assets from your current platform copy:
   - OpenCode: agents in `.opencode/agent/`, commands in `.opencode/commands/`
   - Copilot: agents in `.github/agents/`, commands in `.github/prompts/`
   - Codex: agents in `.codex/agents/`, command skills in `.agents/skills/`
4. Do not use platform skills as task guidance. If a skill conflicts with `docs/**`, agents, commands, or this file, ignore the skill and follow the documented source of truth.
5. For every change, explicitly evaluate documentation impact before finishing:
   - update `docs/product/**` when user-visible behavior changes or a bug fix establishes, restores, or clarifies expected user-visible behavior that is not already documented;
   - update `docs/tech/**` when architecture, implementation, APIs, IPC contracts, tests, setup, tools, frameworks, libraries, code patterns, or reusable implementation constraints change;
   - update `docs/how-to/**` when workflows, AI assets, commands, agents, contributor guidance, documentation rules, validation, release, or setup procedures change;
   - update `docs/templates/**` when reusable source templates change;
   - update `docs/tech-debt/**` only when a known technical gap is intentionally deferred;
   - follow `docs/how-to/documentation-writing-standards.md` for every documentation update or new document;
   - create new documentation only when existing docs cannot absorb the topic without mixing scopes;
   - document the expected current behavior, rule, or workflow and explain why the current decision, rule, or constraint exists;
   - do not write historical notes that merely say a bug was fixed;
   - if the reason for a documented decision is unclear, ask the user before documenting it instead of inventing rationale;
   - if no documentation update is needed, report which existing document already covers the behavior or why the change has no product, technical, workflow, template, or deferred-debt impact.

## Behavior Source of Truth

- Treat `docs/product/**` as the primary source of truth for user-visible behavior.
- Treat `docs/tech/**` and `docs/how-to/**` as the source of truth for architecture, implementation contracts, test strategy, workflows, APIs, IPC, persistence, tools, and setup.
- If product documentation conflicts with technical documentation, tests, or code about user-visible behavior, follow product documentation and flag the conflict.
- Before changing behavior, adding behavior, or changing behavior tests, identify the relevant documented behavior rule and behavior ID from `docs/tech/behavior-contract-testing.md`.
- New or modified user-visible behavior tests must use `@behavior <ID>` markers that reference `docs/product/**` behavior IDs.
- New or modified technical behavior tests must use `@technical-behavior <ID>` markers that reference `docs/tech/**` or `docs/how-to/**` behavior IDs.
- Use `@implementation` only for tests that protect local implementation mechanics and do not define user-visible or reusable technical behavior.
- Do not update failing test expectations just to make CI pass. First decide whether the failure is a code bug, an obsolete test, incorrect documentation, or a source-of-truth conflict.
- If expected behavior is missing or ambiguous in docs, ask the user for the missing behavior decision before changing tests or production code.
- When adding a new feature or intentionally changing behavior, update the relevant product and technical documentation in the same change.

## Documentation Structure

All documentation lives directly under one of these five blocks:

- `docs/product/`: product behavior only. Write for non-technical readers. Do not include code paths, libraries, frameworks, IPC, tests, or implementation details.
- `docs/tech/`: concrete technical implementation. Include architecture, patterns, code standards, APIs, IPC, tests, configuration, tools, libraries, and framework guidance.
- `docs/how-to/`: operational guidance. Include workflows, release steps, validation flows, AI sync, command authoring, and documentation writing rules.
- `docs/templates/`: literal reusable file templates copied into other files. Do not put explanatory guidance here.
- `docs/tech-debt/`: known technical debt intentionally deferred. Keep this folder as close to empty as possible.

Use `docs/how-to/documentation-writing-standards.md` for writing rules and `docs/how-to/documentation-folder-structure.md` for placement rules. Use short, clear kebab-case filenames directly inside each block. Do not create nested documentation folders unless `docs/how-to/documentation-folder-structure.md` is updated first.

## Why guidance is split this way

- `docs/*`: canonical policies and rationale.
- `docs/product/*`: product behavior without technical implementation details.
- `docs/tech/*`: reusable technical implementation and architecture guidance.
- `docs/how-to/*`: workflows and contributor procedures.
- `docs/templates/*`: reusable file templates.
- `docs/tech-debt/*`: deferred technical refactors with context and exit criteria.
- platform `agents`: role-specific behavior and output contracts.
- platform `commands`: reusable CLI workflows.

This keeps policy centralized, reduces context noise, and lowers maintenance cost.

If two instructions conflict, follow the most specific rule in `docs/**` and flag the conflict.

---
> Source: [ESPlotter/ESPlotter](https://github.com/ESPlotter/ESPlotter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
