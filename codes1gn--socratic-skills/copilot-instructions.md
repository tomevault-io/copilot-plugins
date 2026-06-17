## socratic-skills

> This skill teaches AI agents how to use `sk` — the skills manager for AI agent skill packs.

# socratic-skills — sk CLI Skill

**Skill:** `sk`  
**Version:** 1.0.0  
**Platforms:** GitHub Copilot (VS Code), Cursor IDE

This skill teaches AI agents how to use `sk` — the skills manager for AI agent skill packs.

---

## Quick Start

```bash
# Install all skills to both Cursor and Copilot
sk install-all

# List available skills
sk list

# Install a specific skill
sk install arxiv-cli

# Run integration tests
sk test

# Run batch subagent tests (8,400 checks)
sk test --batch
```

---

## Commands Reference

| Command | Description |
|---------|-------------|
| `sk list` | List all registered skills with version + description |
| `sk info <skill>` | Show detailed info: repo, platforms, CLI entry, SKILL.md path |
| `sk install <skill>` | Install skill to `~/.cursor/skills/` and `~/.copilot/skills/` |
| `sk install-all` | Install all 4 skills to all platforms |
| `sk update` | Pull latest from all git submodules |
| `sk search <query>` | Search skills by name, description, or tag |
| `sk test [skill]` | Run integration tests for skill(s) |
| `sk test --batch` | Batch mode: 8 workers × 10 runs |

---

## Platform Compatibility

| Feature | GitHub Copilot | Cursor IDE |
|---------|---------------|------------|
| `sk` CLI | ✅ integrated terminal | ✅ terminal |
| Install skills | ✅ `~/.copilot/skills/` | ✅ `~/.cursor/skills/` |
| Pattern tests | ✅ | ✅ |
| Batch testing | ✅ | ✅ |

---

## Skills Registry

The `sk` CLI manages 4 skills:

| Skill | CLI | Description |
|-------|-----|-------------|
| `agent-handoff` | `handoff`/`resume` | Cross-session memory and handoff |
| `arxiv-cli` | `arxiv`/`ax` | arXiv paper search and download |
| `workflows` | `wf` | Multi-agent workflow orchestration |
| `ir-explorer` | `irx` | LLVM/MLIR IR visualizer |

---

## Installation

```bash
# From source
git clone git@github.com:codes1gn/socratic-skills.git
cd socratic-skills
pip install -e .

# Install all skills
sk install-all
```

---

## File Locations

```
~/.cursor/skills/<skill>/SKILL.md    # Cursor
~/.copilot/skills/<skill>/SKILL.md   # GitHub Copilot
~/.claude/skills/<skill>/SKILL.md    # Claude (future)
```

---

## Agent Tips

1. Run `sk list` to see what skills are available and their versions
2. Run `sk test` after `sk update` to verify skills still work
3. Use `sk info <skill>` to find the SKILL.md path before reading it
4. Use `sk search <query>` to find skills by capability (e.g., `sk search llvm`)

---
> Source: [codes1gn/socratic-skills](https://github.com/codes1gn/socratic-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
