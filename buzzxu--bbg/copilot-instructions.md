## bbg

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**bbg** (BadBoy Genesis) is a CLI tool that generates and maintains AI development workflow governance scaffolding for mono-repo and multi-repo projects. It produces agent instructions, rules, hooks, commands, skills, and MCP configurations for 6 AI coding tools: Claude Code, OpenCode, Cursor, Codex CLI, GitHub Copilot, and Kiro.

**Version:** 0.1.0  
**License:** AGPL-3.0  
**Runtime:** Node.js >= 18 (ESM-only)

## Source Architecture

```
src/
├── cli.ts                    # CLI entry point (Commander + Inquirer)
├── constants.ts              # Shared constants and template manifest
├── analyzers/                # 4 deterministic project analyzers
│   ├── detect-stack.ts       # Language, framework, package manager detection
│   ├── detect-structure.ts   # Monorepo vs polyrepo structure detection
│   ├── detect-deps.ts        # Dependency analysis
│   ├── detect-testing.ts     # Test framework detection
│   └── index.ts              # Analyzer barrel export
├── commands/                 # 6 CLI commands
│   ├── init.ts               # Initialize governance in a project
│   ├── add-repo.ts           # Add a repository to governance
│   ├── doctor.ts             # Health check for governance setup
│   ├── sync.ts               # Sync governance across repos
│   ├── release.ts            # Release management
│   └── upgrade.ts            # Upgrade governance templates
├── config/                   # Configuration management
├── doctor/                   # Doctor checks and fix logic
│   ├── checks.ts             # Governance health checks
│   └── fix.ts                # Auto-fix for detected issues
├── templates/                # Template rendering engine
│   └── render.ts             # Handlebars template rendering
├── upgrade/                  # Template upgrade diffing
│   └── diff.ts               # Template version comparison
└── utils/                    # Shared utilities
    ├── fs.ts                 # Filesystem helpers (exists, readJson, etc.)
    └── logger.ts             # Logger utility

templates/                    # 2-tier template system
├── generic/                  # Verbatim copy templates
└── handlebars/               # Rendered with project context
    ├── AGENTS.md.hbs         # Root AGENTS.md template
    ├── child-AGENTS.md.hbs   # Per-repo AGENTS.md template
    ├── README.md.hbs         # README template
    └── ...                   # Git hooks, scripts, docs
```

## Governance Scaffold Architecture

```
agents/                       # 25 agent definitions
├── Core (8):                 planner, architect, tdd-guide, code-reviewer,
│                             security-reviewer, build-error-resolver,
│                             refactor-cleaner, e2e-runner
├── Support (5):              doc-updater, loop-operator, harness-optimizer,
│                             database-reviewer, devops-reviewer
├── Language (6):             typescript-reviewer, python-reviewer, go-reviewer,
│                             java-reviewer, rust-reviewer, kotlin-reviewer
└── Build Resolvers (6):      typescript-build-resolver, python-build-resolver,
                              go-build-resolver, java-build-resolver,
                              rust-build-resolver, cpp-build-resolver

skills/                       # 61 skill directories (each with SKILL.md)
├── Core (21):                coding-standards, tdd-workflow, security-review,
│                             verification-loop, search-first, writing-plans,
│                             continuous-learning,
│                             eval-harness, strategic-compact, api-design,
│                             backend-patterns, database-migrations, postgres-patterns,
│                             frontend-patterns, e2e-testing, deployment-patterns,
│                             docker-patterns, kubernetes-patterns, harness-engineering,
│                             autonomous-loops, mcp-integration
├── Language (24):            typescript-patterns, react-patterns, nextjs-patterns,
│                             vue-patterns, python-patterns, python-testing,
│                             django-patterns, fastapi-patterns, golang-patterns,
│                             golang-testing, gin-patterns, java-patterns,
│                             springboot-patterns, springboot-testing, jpa-patterns,
│                             rust-patterns, rust-testing, axum-patterns,
│                             kotlin-patterns, android-patterns, ktor-patterns,
│                             laravel-patterns, laravel-testing, cpp-patterns
└── Operations (16):          ci-cd-patterns, monitoring-patterns, git-workflow,
                              code-review-checklist, incident-response,
                              performance-optimization, security-hardening,
                              dependency-audit, secrets-management,
                              microservices-patterns, event-driven-architecture,
                              data-modeling, api-versioning, prompt-engineering,
                              llm-cost-optimization, agent-orchestration

rules/                        # 34 rule files
├── common/ (8):              coding-style, git-workflow, testing, security,
│                             performance, patterns, hooks, agents
├── typescript/ (5):          coding-style, testing, react, node, security
├── python/ (4):              coding-style, testing, django, security
├── golang/ (4):              coding-style, testing, patterns, security
├── java/ (4):                coding-style, testing, spring, security
├── rust/ (3):                coding-style, testing, security
├── kotlin/ (3):              coding-style, testing, security
└── php/ (3):                 coding-style, testing, security

commands/                     # 40 slash commands
├── Core (10):                plan, tdd, code-review, build-fix, security-scan,
│                             refactor-clean, e2e, test-coverage, update-docs, doctor
├── Learning (6):             learn, learn-eval, checkpoint, verify, sessions, eval
├── Language (16):            ts-review, ts-build, ts-test, python-review,
│                             python-build, python-test, go-review, go-test,
│                             go-build, java-review, java-build, java-test,
│                             rust-review, rust-build, rust-test, kotlin-review
└── Orchestration (8):        orchestrate, loop-start, loop-status, quality-gate,
                              harness-audit, model-route, setup-pm, sync

hooks/                        # Hook automation
├── hooks.json                # Hook event configuration
├── README.md                 # Hook documentation
└── scripts/                  # 6 automation scripts
    ├── session-start.js      # Load session context
    ├── session-end.js        # Save session state
    ├── pre-edit-check.js     # Check for debug/secrets before edits
    ├── post-edit-typecheck.js # Auto-typecheck after .ts edits
    ├── security-scan.js      # Detect secrets, block destructive commands
    └── suggest-compact.js    # Token usage tracking

mcp-configs/                  # MCP server configurations
├── mcp-servers.json          # 12 server configs (github, filesystem, postgres,
│                               sqlite, brave-search, puppeteer, memory, fetch,
│                               sequential-thinking, context7, supabase, redis)
└── README.md                 # Setup guide

contexts/                     # Operational contexts
├── dev.md                    # Development mode
├── review.md                 # Code review mode
└── research.md               # Research/exploration mode
```

## Build & Test Commands

```bash
# Build
npm run build          # tsup → dist/cli.js

# Development
npm run dev            # tsx src/cli.ts (direct execution)

# Test
npm run test           # vitest run (all tests)
npm run test:watch     # vitest (watch mode)

# CLI Usage
npx bbg init           # Initialize governance
npx bbg add-repo       # Add a repository
npx bbg doctor         # Health check
npx bbg sync           # Sync governance
npx bbg release        # Release management
npx bbg upgrade        # Upgrade templates
```

## Key Design Decisions

- **ESM-only**: All imports use `.js` extensions for Node.js ESM compatibility
- **2-tier templates**: generic (verbatim copy), handlebars (rendered with project variables)
- **4 analyzers**: detect-stack, detect-structure, detect-deps, detect-testing
- **Supported stacks**: Node.js, Java (Maven/Gradle), Python, Go
- **Template manifest**: All template paths are declared in `src/constants.ts`
- **Shared governance**: Agents, skills, rules, and commands are shared across all 6 AI tool configs

## Development Notes

- File naming: lowercase with hyphens (e.g., `detect-stack.ts`, `add-repo.ts`)
- Template rendering uses Handlebars with `{{variable}}` syntax
- `import ... with { type: "json" }` syntax is used for JSON imports (Node 18+)
- All async functions use proper error handling with try/catch

## Code Style

- Prefer immutable data: create new objects rather than mutating
- Small focused functions (<50 lines recommended)
- TypeScript strict mode enabled
- Use `const` over `let`, never use `var`
- Handle all errors explicitly, never silently swallow
- Use early returns to reduce nesting

## Working with Agents

When performing tasks, reference the appropriate agent definition:

| Task | Agent | Skill |
|------|-------|-------|
| Planning | `agents/planner.md` | — |
| Architecture | `agents/architect.md` | — |
| TDD | `agents/tdd-guide.md` | `skills/tdd-workflow/SKILL.md` |
| Code Review | `agents/code-reviewer.md` | `skills/code-review-checklist/SKILL.md` |
| Security | `agents/security-reviewer.md` | `skills/security-review/SKILL.md` |
| Build Errors | `agents/build-error-resolver.md` | `skills/verification-loop/SKILL.md` |
| Refactoring | `agents/refactor-cleaner.md` | `skills/coding-standards/SKILL.md` |
| TypeScript | `agents/typescript-reviewer.md` | `skills/typescript-patterns/SKILL.md` |

## Slash Commands

Use these commands for common workflows:

- `/plan` — Create implementation plan
- `/tdd` — Test-driven development cycle
- `/code-review` — Full code review
- `/build-fix` — Fix build errors
- `/security-scan` — Security audit
- `/doctor` — Governance health check
- `/verify` — Verification loop

See `commands/*.md` for all 32 available commands.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buzzxu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/buzzxu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
