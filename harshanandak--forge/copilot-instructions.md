## forge

> A TDD-first workflow for AI coding agents. Ship features with confidence.

# Forge - 7-Stage TDD Workflow

A TDD-first workflow for AI coding agents. Ship features with confidence.

## Commands (7 Stages)

| Stage | Command | Description |
|-------|---------|-------------|
| utility | `/status` | Check current context, active work, recent completions |
| 1 | `/plan` | Design intent Q&A → research → branch + task list |
| 2 | `/dev` | Subagent-driven TDD per task (spec + quality review) |
| 3 | `/validate` | Validation (type/lint/security/tests) |
| 4 | `/ship` | Create PR with full documentation |
| 5 | `/review` | Address ALL PR feedback |
| 6 | `/premerge` | Complete docs on feature branch, hand off PR to user |
| 7 | `/verify` | Post-merge health check (CI on main) |

## Workflow Flow

```
/plan → /dev → /validate → /ship → /review → /premerge → /verify
```

## Core Principles

- **TDD-First**: Write tests BEFORE implementation (RED-GREEN-REFACTOR)
- **Design-First**: One-question-at-a-time Q&A captures design intent upfront
- **HARD-GATEs**: Every stage exit has explicit pass criteria — run the commands, show the output
- **Security Built-In**: OWASP Top 10 analysis for every feature

## Prerequisites

- Git, GitHub CLI (`gh`)
- Beads (recommended): `bun add -g @beads/bd && bd init`

## Quick Start

1. `/status` - Check where you are
2. `/plan <feature-slug>` - Design intent → research → branch + task list
3. `/dev` - Implement with TDD
4. `/validate` - Validate everything
5. `/ship` - Create PR
6. `/review <pr-number>` - Address all feedback
7. `/premerge <pr-number>` - Docs + hand off to user

## Stage Details

### Utility: Status (`/status`)

Check current context before starting work:
- Active issues (via Beads if installed)
- Recent completions
- Current branch state

### 1. Plan (`/plan <feature-slug>`)

Three phases:
- **Phase 1**: Design intent Q&A (one-question-at-a-time with user)
- **Phase 2**: Technical research (web + codebase, OWASP Top 10)
- **Phase 3**: Create branch + task list (TDD-ordered)

### 2. Development (`/dev`)

Subagent-driven TDD per task:
- Implementer subagent: RED-GREEN-REFACTOR enforced by HARD-GATE
- Spec compliance reviewer: checks every task
- Code quality reviewer: checks after spec compliance
- Decision gate: 7-dimension scoring when spec gap found

### 3. Validate (`/validate`)

Validate everything (HARD-GATE exit — fresh output required):
- Type checking
- Linting (0 errors, 0 warnings)
- All tests passing
- Security scan (OWASP Top 10)

### 4. Ship (`/ship`)

Create pull request:
- Push branch
- Create PR with design doc reference
- Link to Beads issue

### 5. Review (`/review <pr-number>`)

Address ALL feedback:
- GitHub Actions failures
- Greptile inline comments (reply + resolve each)
- SonarCloud issues
- Other CI/CD tool feedback

### 6. Premerge (`/premerge <pr-number>`)

Complete docs and hand off (NEVER merges):
- Update CLAUDE.md, AGENTS.md, GEMINI.md, README as needed
- Commit docs to feature branch
- Hand off PR URL to user for merge

### 7. Verify (`/verify`)

Post-merge health check:
- CI on main: all checks green
- Close Beads issue
- Confirm merge landed

## Directory Structure

```
your-project/
├── AGENTS.md                    # Universal (Windsurf, Cursor, Kilo, OpenCode, Cline, Roo, Aider)
├── CLAUDE.md                    # Claude Code
├── GEMINI.md                    # Google Antigravity
├── .cursorrules                 # Cursor
│
├── .claude/commands/            # Claude Code commands
└── docs/
    ├── plans/
    │   ├── YYYY-MM-DD-<slug>-design.md
    │   └── YYYY-MM-DD-<slug>-tasks.md
    └── TOOLCHAIN.md
```

## Supported Agents

This workflow works with ALL major AI coding agents:

| Agent | Instructions | Commands |
|-------|-------------|----------|
| Claude Code | CLAUDE.md | .claude/commands/ |
| Google Antigravity | GEMINI.md | .agent/workflows/ |
| Cursor | .cursorrules | .cursor/rules/ |
| Windsurf | AGENTS.md | .windsurf/workflows/ |
| Kilo Code | AGENTS.md | .kilocode/workflows/ |
| OpenCode | AGENTS.md | .opencode/commands/ |
| Cline | AGENTS.md | - |
| Roo Code | AGENTS.md | .roo/commands/ |
| Continue | AGENTS.md | .continue/prompts/ |
| GitHub Copilot | .github/copilot-instructions.md | .github/prompts/ |
| Aider | AGENTS.md (via .aider.conf.yml) | In-chat |
| Codex CLI | AGENTS.md | In-chat |

## License

MIT

---

See `AGENTS.md` for the complete workflow guide.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harshanandak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
