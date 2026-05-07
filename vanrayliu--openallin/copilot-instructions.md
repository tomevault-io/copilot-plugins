## openallin

> > A layered, composable engineering framework combining the best of OpenSpec, Superpowers, GSD, OMC, ECC, and Trellis.

# OpenAllIn — Unified AI Coding Harness

> A layered, composable engineering framework combining the best of OpenSpec, Superpowers, GSD, OMC, ECC, and Trellis.

## Architecture

OpenAllIn uses a **layered, composable** design where each layer is independently optional and freely combinable:

```
┌─────────────────────────────────────────────────┐
│  Layer 6: Workspace  — Project Memory &          │  ← Trellis
│             Parallel Pipeline                     │
├─────────────────────────────────────────────────┤
│  Layer 5: Orchestration — Multi-Agent Team       │  ← OMC
│             Coordination                          │
├─────────────────────────────────────────────────┤
│  Layer 4: Execution — Phase-Based Workflow &     │  ← GSD
│             Context Engineering                   │
├─────────────────────────────────────────────────┤
│  Layer 3: Enhancement — Security / Memory /      │  ← ECC
│             Learning / Verification               │
├─────────────────────────────────────────────────┤
│  Layer 2: Skills — Engineering Methodology       │  ← Superpowers
│             Library                               │
├─────────────────────────────────────────────────┤
│  Layer 1: Spec — Spec-Driven Development         │  ← OpenSpec
└─────────────────────────────────────────────────┘
```

## Core Principles

1. **Spec First** — Align on requirements before writing code
2. **Engineering Discipline** — Make best practices the default behavior
3. **Context Hygiene** — Small tasks, phase separation, structured information
4. **Team Orchestration** — Multiple agents collaborate like a real team
5. **Capability Enhancement** — Security, memory, learning, verification loops
6. **Project Memory** — File-based persistence over LLM memory

## Installation

### Quick Install

```bash
# Clone and install
git clone https://github.com/vanrayliu/openallin.git
cd openallin
bash scripts/install.sh opencode   # For OpenCode
bash scripts/install.sh claude     # For Claude Code
bash scripts/install.sh all        # For all tools
```

### OpenCode

```bash
# Project-level
cp AGENTS.md your-project/
cp project.md your-project/
cp -r rules/ your-project/.opencode/rules/
cp -r agents/ your-project/.opencode/agents/

# Skills need SKILL.md wrapper — use install script instead
# Or manually: for each skill, create .opencode/skills/<name>/SKILL.md with frontmatter

# Or use the install script
bash scripts/install.sh opencode your-project/
```

OpenCode reads `AGENTS.md` natively. Skills go to `.opencode/skills/<name>/SKILL.md` with YAML frontmatter.

### Claude Code

```bash
# Project-level
cp AGENTS.md your-project/CLAUDE.md
cp project.md your-project/
cp -r rules/ your-project/.claude/rules/
cp -r agents/ your-project/.claude/agents/

# Skills need SKILL.md wrapper — use install script instead
# Or manually: for each skill, create .claude/skills/<name>/SKILL.md with frontmatter

# Hooks
cp -r hooks/ your-project/.claude/hooks/
# Then configure hooks in .claude/settings.json (see README.md)

# Or use the install script
bash scripts/install.sh claude your-project/
```

Claude Code reads `CLAUDE.md` natively. Hooks are configured in `.claude/settings.json`.

### Other Tools

| Tool | Install | Notes |
|------|---------|-------|
| **Cursor** | Copy `AGENTS.md` to project root or `.cursor/rules/` | Reads `.mdc` files |
| **Codex** | Copy `AGENTS.md` to project root | Reads AGENTS.md |
| **Gemini CLI** | Copy `AGENTS.md` to project root | Claude Code compatible |
| **Windsurf** | Copy `AGENTS.md` to project root | Auto-detects |

### Minimal Install (Universal)

```bash
cp AGENTS.md your-project/
cp project.md your-project/
cp -r specs/ your-project/
cp -r changes/ your-project/
cp -r workspace/ your-project/workspace/
```

This works with any tool that reads `AGENTS.md` from the project root.

## Quick Start

### Natural Language Commands

You can use **natural language** to trigger these actions:

| You say | AI executes |
|---------|-------------|
| "Install OpenAllIn" | `bash scripts/install.sh` |
| "Update OpenAllIn" | `bash scripts/update.sh` |
| "Uninstall OpenAllIn" | `bash scripts/uninstall.sh` |
| "I want to add a feature" | `/oa-propose <name>` |
| "Discuss requirements" | `/oa-discuss` |
| "Start implementation" | `/oa-apply <name>` |
| "Check code quality" | `/oa-review` |
| "Security check" | `/oa-security` |
| "Ship this" | `/oa-ship` |
| "Deploy to production" | `/oa-land` |
| "Debug this bug" | `/oa-debugging` |
| "Write a test" | `/oa-tdd` |

---

### Spec-Driven Workflow

```
/oa-propose <name>      → Create change proposal
/oa-apply <name>        → Execute tasks from checklist
/oa-validate <name>     → Validate spec format
/oa-archive <name>      → Archive change and merge to specs
/oa-discuss             → Discuss and clarify requirements (multi-round)
/oa-plan                → Plan atomic tasks
/oa-execute             → Execute in waves
/oa-verify              → Verify quality
/oa-ship                → Ship and create PR
/oa-review              → Code/design/architecture review
/oa-ui-design           → UI/UX design intelligence system
/oa-security            → Security audit (OWASP + STRIDE)
/oa-land                → Deployment verification
/oa-qa-browser          → Browser testing (Playwright)
/oa-benchmark           → Performance testing
/oa-team-plan           → Team planning (multi-agent)
/oa-team-exec           → Team execution (parallel worktree)
/oa-team-verify         → Team verification (review + security)
/oa-brainstorming       → Brainstorming session (multi-round)
/oa-debugging           → Systematic debugging (multi-round)
/oa-tdd                 → Test-driven development (Red-Green-Refactor)
/oa-writing-plans       → Plan writing (multi-round)
/oa-worktree            → Git worktree isolation
```

---

## Script Commands (via Natural Language)

These commands are scripts, not skills. Use natural language to trigger them:

| You say | Script |
|---------|--------|
| "Install OpenAllIn" | `bash scripts/install.sh` |
| "Update OpenAllIn" | `bash scripts/update.sh` |
| "Uninstall OpenAllIn" | `bash scripts/uninstall.sh` |
| "Preview update" | `bash scripts/update.sh --dry-run` |
| "Force update" | `bash scripts/update.sh --force` |

---

## License

MIT

---
> Source: [vanrayliu/openallin](https://github.com/vanrayliu/openallin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
