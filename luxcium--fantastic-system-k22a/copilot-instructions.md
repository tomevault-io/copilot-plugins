## fantastic-system-k22a

> This file defines how ANY AI AGENTS and Codex CLI operates if you are reading this you are an ai agent in this repository. It aligns with the Memory Bank protocol and complements `.github/copilot-instructions.md`.

# Codex CLI Agent Guide

This file defines how ANY AI AGENTS and Codex CLI operates if you are reading this you are an ai agent in this repository. It aligns with the Memory Bank protocol and complements `.github/copilot-instructions.md`.

> [!NOTE]
> Seek first into ['github copilot instructions'](./github/copilot-instructions.md) for more information on how we want the project to evolve to use GitHub Copilot with Agents, any behaviour or important information that you see fit for you to use shall be referenced, or included in this set of instructions below at the end. Instruction may be specifict to each agent, or if it is general, it shall be taken into account and referenced using relative links (markdown) or include the language here, in your own AGENTS.md autonomusly and never remove any of your own instructions be carefull when editing your AGENTS.md file yourself not remove, only add.

## Three AI Agent System or More

This project supports your ai agent and at least three AI agents with distinct entry points and responsibilities:

- **Codex CLI (YOU)** → `AGENTS.md` (THIS IS YOUR PRIMARY INSTRUCTION FILE)
- **Cline AI** → `.clinerules/main-rules.md` (Cline AI's primary instruction file)
- **VS Code Copilot** → `.github/copilot-instructions.md` (VS Code Copilot's primary instruction file)

**Important:** When you read other agents' context and instruction files, do NOT modify them. Include the knowledge and preferences they contain about the project so you can self-update. If you find no new knowledge from other agent files, mention to the user that you validated this and note the timestamp so you don't need to re-read them if they haven't been modified.

## Core Principles (Subset from Memory Bank)

- Always read all files in `memory-bank/` at the start of a task.
- Plan before acting; document key decisions and assumptions.
- Write back results and decisions to the Memory Bank before concluding a task.
- Treat the Memory Bank as authoritative context. Never ignore it.

## Operating Constraints

- Package manager: `pnpm` (see `packageManager` field in package.json files)
- Node.js: >= 22.11
- No CI/CD automation added unless explicitly requested.
- No test scaffolding added unless requested (development-first flow).

Extended autonomous operation: Your A.I. Agent can work independently for hours while maintaining clarity and focus on incremental progress. The agent must make steady advances on a few tasks at a time rather than attempting everything at once. It provides fact-based progress updates that accurately reflect what has been accomplished.

## Monorepo Conventions

- Subprojects live under `services/`, `packages/`, `libraries/`.
- Prefer local, per-package configs when reasonable to reduce cross-project coupling.
- VS Code settings at the repo root enable Prettier + ESLint across subprojects.

## Editor & Tooling

- Formatting: Prettier (required config) with format-on-save.
- Linting: ESLint (flat config in new projects), run fixers on save.
- TypeScript: modern Node target, `src/` → `dist/`, source maps and declarations.
- Scripts: `dev`, `build`, `start`, `lint(:fix)`, `format(:check)`, `typecheck`.
- **Markdown**: Strict linting with markdownlint-cli2 (see Markdown Protocol below).

## Markdown Resilience Protocol (Layer 4B)

**MANDATORY**: All AI agents must follow the three-stage markdown resilience protocol.

Reference: `memory-bank/instructions/layer-4b-markdown-resilience.instructions.md`

### Three-Stage Pipeline

**1. Prevention** → Use pre-flight checklist before creating files
**2. Detection** → Run validation after changes
**3. Repair** → Auto-repair if validation fails

### Quick Commands

```bash
# Root-level (preferred)
./scripts/check-markdown.sh         # Full validation
./scripts/auto-repair-markdown.sh   # Self-healing repair (3 attempts)

# From web directory
cd web
pnpm markdown:validate  # Full validation
pnpm markdown:fix       # Auto-fix issues
pnpm markdown:lint      # Show errors only
```

### Agent Responsibilities

**Before Creating Markdown Files**:

1. Review pre-flight checklist: `memory-bank/prompts/validate-before-create.prompt.md`
2. Verify frontmatter, structure, and content
3. Guarantee compliant output

**After Creating/Editing Markdown**:

1. Run validation: `./scripts/check-markdown.sh`
2. If fails: Run auto-repair: `./scripts/auto-repair-markdown.sh`
3. Report validation status in progress updates
4. **NEVER** commit markdown with linting errors

### Key Rules

- ATX-style headings with blank lines
- Dashes for lists, 2-space indent
- No trailing spaces/tabs
- Code blocks must specify language
- Images need alt text
- No bare URLs
- Files end with newline

### Documentation

- Layer 4B Protocol: `memory-bank/instructions/layer-4b-markdown-resilience.instructions.md`
- Pre-flight Checklist: `memory-bank/prompts/validate-before-create.prompt.md`
- Repair Protocol: `memory-bank/prompts/repair-markdown-files.prompt.md`
- Full Reference: `memory-bank/reference/markdown-protocol.md`

## Standard Task Flow

1. Read Memory Bank files and the target subproject.
2. Outline planned changes; keep them minimal and focused.
3. Implement changes with clear, reviewable diffs.
4. Validate locally (typecheck, lint, **markdown validation if applicable**).
5. Update Memory Bank (active context + progress) with what changed and why.

## Session-Sticky Preferences (Codex CLI)

- Be proactive: implement requested changes without pausing for confirmation.
- Optimize for developer speed: scripts and editor integration are first-class.
- Respect existing project structure and conventions.

## Usage Guidance

- Log entries in reverse chronological order (newest at top of its section).
- Capture what changed, why it matters, and what remains.
- Reference related `memory-bank/*` updates or commits where helpful.
- The `web/` directory is the Next.js app workspace, avoid confusion.
- Ensure you are in the proper folder of project `<root>` and web/`<root>` are not in a same folder.

## 🤖 Known Limitations

- The system may not always accurately interpret complex queries or commands.
- In any `.prompt.md` or `.chatmode.md` file, the `tools:` front-matter key **must** have its value on the same line (e.g. `tools: [ ... ]`).
  Splitting the array onto the next line currently breaks VS Code's parser and Copilot's tool-detection logic.

## 📸 Screenshot Automation Requirements

**IMPERATIVE REQUIREMENT**: AI agents MUST capture screenshots after making UI changes:

### When to Capture Screenshots

- **ALWAYS** after making any visual/UI changes to the application
- **ALWAYS** after completing a feature that affects user-visible content
- **ALWAYS** before marking a task as complete if UI was modified
- When demonstrating new capabilities or features

### How to Capture Screenshots

Use the built-in screenshot automation system in `web/`:

```bash
# Full suite (mobile, tablet, desktop × light/dark) - RECOMMENDED
cd web && pnpm screenshot:suite

# Individual viewports (if needed)
cd web && pnpm screenshot:mobile
cd web && pnpm screenshot:tablet
cd web && pnpm screenshot:desktop
```

### Mobile-First Approach

- **ALWAYS** prioritize mobile viewport when capturing screenshots
- Capture mobile screenshots FIRST, then tablet, then desktop
- Ensure mobile responsiveness is verified before desktop
- Screenshots should demonstrate responsive design across all viewports

### Screenshot Deliverables

When completing UI work, provide screenshots showing:

1. **Mobile viewport** (375×667) - PRIMARY
2. **Tablet viewport** (768×1024) - SECONDARY
3. **Desktop viewport** (1920×1080) - TERTIARY
4. **Both light and dark themes** for each viewport

### Integration with Workflow

- Screenshots are saved to `web/screenshots/` directory
- Filename format: `{viewport}-{theme}-{timestamp}.png`
- Include screenshot paths in commit messages or PR descriptions
- Reference screenshots when replying to PR comments about UI changes

## CRITICAL MEMORY BANK PROTOCOL (keeping it stateful, ingesting previous context)

**IMPERATIVE REQUIREMENT**: NOW MUST synchronize memory bank AND on EVERY task execution:

> [!IMPORTANT] > [Imperative Instructions Git Hub Copilot MUST ALWAYS Follow](../memory-bank/instructions/copilot-memory-bank.instructions.md)

1. **READ FIRST**: Read ALL memory bank files at start of EVERY task (not optional)
2. **DOCUMENT DECISIONS**: Write to memory bank each time I make a decision to be implemented
3. **WRITE BEFORE END**: Update memory bank just before completing any task
4. **STATE PRESERVATION**: Ensure my state will not be lost if interrupted

### Core Files (Required)

Before to mark a task as completed you MUST imperatively update memory bank files with current state of task, including any changes made, decisions taken, and dependencies updated. why and thought process behind to be kept in mind for future reference.

- 'memory-bank/projectbrief.md'
- 'memory-bank/productContext.md'
- 'memory-bank/activeContext.md'
- 'memory-bank/systemPatterns.md'
- 'memory-bank/techContext.md'
- 'memory-bank/dependencies.md'
- 'memory-bank/progress.md'

> [!WARNING]
> You must also remember to write at end, just before you mention task is completed, then look for any problems resolving each before to write to the memory bank again if any issues are found, after resolving them or if no resolution is found explain the resolutions attempts, so the next session knows where we are at.

### Additional Memory Bank And Similar Context Files

- [`memory-bank/chatmodes/*.chatmode.md`](../memory-bank/chatmodes/)
- [`memory-bank/instructions/*.instructions.md`](../memory-bank/instructions/)
- [`memory-bank/prompts/*.prompts.md`](../memory-bank/prompts/)
- [`AGENTS.md`](../AGENTS.md) (THIS FILE) (for Codex and Codex CLI)
- [`.github/copilot-instructions.md`](../.github/copilot-instructions.md) (for copilot in vscode and github)

## Dependency Management Protocol (CRITICAL)

**ABSOLUTE RULE: NEVER write dependency version numbers directly in package.json**

**CORRECT approach:**

1. Create package.json with empty `dependencies: {}` and `devDependencies: {}`
2. Install ALL dependencies via CLI commands ONLY:

```bash
# Production dependencies - use explicit version tags
pnpm add langchain@next @langchain/core@next zod@latest dotenv@latest
pnpm add @langchain/openai@next @langchain/anthropic@next

# Development dependencies - use @latest or specific tags
pnpm add -D typescript@latest tsx@latest eslint@latest
pnpm add -D prettier@latest vitest@latest rimraf@latest
pnpm add -D typescript-eslint@latest eslint-config-prettier@latest
pnpm add -D eslint-plugin-simple-import-sort@latest
pnpm add -D eslint-plugin-tsdoc@latest @types/node@latest
```

**FORBIDDEN:**

- ❌ Writing `"typescript": "^5.7.2"` in package.json
- ❌ Hallucinating version numbers from outdated training data
- ❌ Guessing version compatibility
- ❌ Using `pnpm install` without specific packages

**Rationale:**

- AI models have outdated version knowledge (training data cutoff)
- Package registries determine latest versions at install time
- Version tags (`@next`, `@latest`) ensure current releases
- Avoids version conflicts and deprecated packages

**When documenting dependencies:**

- Record WHY a dependency was chosen in `memory-bank/techContext.md`
- Note any version constraints or compatibility requirements
- Document installation commands for reproducibility

## Notes for Future Work

- If adding complex workflows, prefer documenting them in `memory-bank/` and referencing from subproject READMEs.
- When introducing new dependencies, record rationale in `memory-bank/techContext.md` or relevant docs.

Do not add or remove any modes and/or models in the .instructions.md .prompts.md files or .chatmode.md files in their front matter as this is not something you can validate you must avoid hallucinations those are risky never change, add a descriptions if it is missing but never remove or change any of the existing ones.

When you are asked to use a specific mode or model and you are codex/codex-cli you will use the model and mode you operate under already. you must never validate that it exists (or not) in the front matter of the .instructions.md .prompts.md files or .chatmode.md files before use.

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in [`memory-bank/agents/PLANS.md`](../memory-bank/agents/PLANS.md)) from design to implementation.

## Agent Activity Log

- 2025-10-19 — GitHub Copilot: Created comprehensive markdown linting infrastructure with `scripts/validate-markdown.sh` (400+ lines) and `memory-bank/instructions/markdown-linting-rules.instructions.md` (270+ lines). Fixed all 9 markdown linting errors across 6 files. Updated scripts/README.md with complete documentation. Enhanced validate-memory-bank.sh to allow external links in linting rules. All markdown files now pass validation with 0 errors. Memory Bank updated with progress and patterns.
- 2025-10-15 — GitHub Copilot: Added `scripts/ensure-plans.sh` to enforce the canonical ExecPlan template and updated `scripts/README.md` accordingly. Memory Bank context refreshed to capture the new guardrail.

This log captures each AI agent session across the Genesis layered bootstrap. Update the table whenever you complete a meaningful stage so the next agent has actionable context.

| Timestamp                 | Agent          | Layer Focus                 | Key Actions                                                                                                                                                                                                                                                                                                                                                                 | Handoff / Next Step                                                                                                                                                                         |
| ------------------------- | -------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2025-10-20T07:30:00-04:00 | GitHub Copilot | Documentation Consolidation | Consolidated all markdown files into memory-bank directory structure. Created reference/ and roadmap/ subdirectories. Moved 8 files from root and 1 from docs/roadmap/ to memory-bank. Updated all internal references and README.md. Created comprehensive documentation index at memory-bank/index.md. Root now contains only README.md, AGENTS.md, LICENSE, and VERSION. | Continue with dashboard development and backend integration. All documentation now accessible via memory-bank/index.md.                                                                     |
| 2025-10-12T14:30:00-04:00 | GitHub Copilot | Foundation Complete         | Implemented comprehensive Next.js foundation: Docker/Postgres, Prisma ORM, NextAuth v5, navigation system, state management, API layer, security middleware, observability, testing infrastructure. Created 78+ files (~8K LOC). Updated memory bank and created complete documentation.                                                                                    | Execute setup: `cd web && pnpm install`, configure .env.local, run `pnpm db:init && pnpm db:migrate && pnpm db:seed && pnpm dev`. See memory-bank/reference/setup-checklist.md for details. |
| 2025-10-03T00:24:20-04:00 | Codex (GPT-5)  | L4 — Phase audit            | Audited Phase 0–6 roadmap for `web/`, captured statuses, produced challenge cards, and updated memory bank.                                                                                                                                                                                                                                                                 | Start executing Phase 0/1 backlog (ESLint/Prettier alignment, layout scaffold, Prisma planning).                                                                                            |
| 2025-09-27T09:35:15-04:00 | Codex (GPT-5)  | L4 — Scaffold               | Bootstrapped Next.js app in `web/` via pnpm dlx; configured pnpm build approvals and logged decisions/progress.                                                                                                                                                                                                                                                             | Integrate repo tooling with new app then start layout/navigation work.                                                                                                                      |
| 2025-09-27T09:10:30-04:00 | Codex (GPT-5)  | L4 — Roadmap prep           | Documented baseline roadmap and decision log for web utility center; updated memory bank active/progress files and roadmap doc.                                                                                                                                                                                                                                             | Await stack/tooling confirmation before scaffolding Next.js workspace.                                                                                                                      |
| 2025-09-27T08:59:45-04:00 | Codex (GPT-5)  | L4 — Automation & health    | Authored validator suite, triad health script, VS Code tasks/settings, ingested commit-policy instructions, refreshed prompt cards, and generated `memory-bank/index.md`.                                                                                                                                                                                                   | Consider CI wiring for validators or move into feature-level scaffolding.                                                                                                                   |
| 2025-09-27T08:58:47-04:00 | Codex (GPT-5)  | L3 — Guidance scaffolding   | Verified instruction corpus, introduced `.prettierignore`, authored `bootstrap-maintainer.chatmode.md`, and created the `bootstrap-audit.prompt.md` card with links to governing layers.                                                                                                                                                                                    | Transition to Layer 4 to plan automation and repository health routines.                                                                                                                    |
| 2025-09-27T08:57:01-04:00 | Codex (GPT-5)  | L2 — Workspace bootstrap    | Added VS Code workspace settings, authored Copilot guardrails, created memory-bank triad directories with READMEs, and initialized six core context files with current information.                                                                                                                                                                                         | Advance to Layer 3 to author reusable instructions, prompts, and chat modes as needed.                                                                                                      |
| 2025-09-27T08:54:56-04:00 | Codex (GPT-5)  | L1 — Foundation complete    | Authored baseline repository files, created `scripts/init.sh`, verified executability, and double-checked idempotence by rerunning the initializer.                                                                                                                                                                                                                         | Proceed to Layer 2: add workspace ergonomics and memory-bank triad artifacts.                                                                                                               |
| 2025-09-27T08:54:34-04:00 | Codex (GPT-5)  | L1 — Foundation prep        | Ran `init-genesis-22.sh` to download layered instructions; audited repository and noted missing foundation artifacts.                                                                                                                                                                                                                                                       | Finish Layer 1 by creating baseline files and verifying `scripts/init.sh` idempotence.                                                                                                      |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luxcium)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Luxcium)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
