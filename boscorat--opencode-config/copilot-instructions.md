## opencode-config

> This file contains both the authoritative contribution rules and high-signal developer guidance to help agents (and humans) work effectively in this repo.

# Agent, Skill, and Command Guidelines

This file contains both the authoritative contribution rules and high-signal developer guidance to help agents (and humans) work effectively in this repo.

## Quick start

Before opening a PR:

```sh
bash .github/workflows/scripts/validate-frontmatter.sh
```

Exit 0 = all artifacts pass. Exit 1 = errors reported. Fix and repeat.

## Developer workflow

### Repository layout

```
agents/               # Agent files (*.md) — one per agent
skills/              # Skill folders — nested by owner or function
  ├─ opencode-agent-expert/     # Top-level if single owner
  ├─ documentation-reviewer/
  │  ├─ doc-analyse/            # Nested skills under one owner
  │  ├─ doc-docstring/
  │  └─ doc-implement/
  └─ python-pyside6-reviewer/
     ├─ py-analyse/
     └─ py-implement/
commands/            # Command files (*.md) — one per command
```

**Key point**: Skills are discovered in priority order:
1. Project-local: `.opencode/skills/<name>/SKILL.md`
2. Global: `~/.agents/skills/<name>/SKILL.md` (symlinked by `install.sh`)

### When to create each artifact

| Artifact | When to create | Scope |
|----------|---|---|
| **Agent** | A new subagent or primary mode agent you want to reuse | Global (`agents/`) or project (`.opencode/agents/`) |
| **Skill** | Specialized instructions that an agent loads on demand (e.g., deep schema reference, workflow guide) | Global (`skills/`) or project |
| **Command** | A shorthand `/name` for a recurring task (e.g., `/new-agent`, `/py-analyse-repo`) | Global (`commands/`) or project |

### Common gotchas

**Skill folder structure matters**: A skill at `skills/opencode-agent-expert/SKILL.md` is discovered as `opencode-agent-expert`, but a skill at `skills/opencode-agent-expert/sub/SKILL.md` is not automatically discovered unless you adjust the skill name to `opencode-agent-expert-sub` and place it at `skills/opencode-agent-expert-sub/SKILL.md`.

**Nested skills under one owner**: If you have multiple skills from the same owner (e.g., `doc-analyse`, `doc-docstring`, `doc-implement` from `documentation-reviewer`), create them as:
```
skills/documentation-reviewer/
  ├─ doc-analyse/SKILL.md         (name: doc-analyse)
  ├─ doc-docstring/SKILL.md       (name: doc-docstring)
  └─ doc-implement/SKILL.md       (name: doc-implement)
```
Each folder must have a `SKILL.md` at the top level with the correct `name` field.

**Deprecated fields**: Always use `permission` instead of the old `tools` block, and `steps` instead of `maxSteps`.

**Permission shapes**: Common patterns:
- Read-only agents (review, audit): must set `bash: deny` and `write: deny`
- PDF-safe agents: include `read: { "anonymised*": "allow", "*.pdf": "ask" }` to guard against sensitive local PDFs
- Task routing: use `permission.task` with specific agent names to prevent self-loops
- Bash glob: for careful commands, use `"ls *": "allow"` instead of `"*": "allow"`

**Permission.task anti-pattern**: Never use `"*": "allow"` in `permission.task`. Always deny your own agent name to prevent self-invocation loops.

### When to update `AGENTS.md`

If you change the contribution rules, **update `AGENTS.md` and the skill body in sync**:

1. Update the rule in this file (the **Contribution rules** section below)
2. Update `skills/opencode-agent-expert/SKILL.md` §6 (anti-pattern index) to match
3. The validator will mirror both sources

This keeps the authoritative rule in one place (`AGENTS.md`) and ensures agents see it when they load the skill.

### Permission matrix quick reference

**Permission block structure**:
```yaml
permission:
  read: allow | ask | deny           # File read access
  glob: allow | ask | deny           # File glob patterns
  grep: allow | ask | deny           # Content search
  bash:
    "*": ask | deny                  # Default for all commands
    "ls *": allow                    # Specific glob patterns
    "test *": allow
  edit: allow | ask | deny           # File editing
  write: allow | ask | deny          # File creation
  webfetch: allow | ask | deny       # URL fetching
  websearch: allow | ask | deny      # Web search
  skill: allow | ask | deny          # Load skills on demand
  task:                              # Subagent delegation
    "*": deny                        # Block all by default
    "explore": allow                 # Name specific agents
    "my-own-name": deny              # NEVER allow self-invocation
  question: allow | ask | deny       # Ask user for input
  external_directory: allow | ask | deny # Access /tmp/opencode
```

---

# Contribution rules

Every PR to this repo must pass the same checks that `opencode-agent-expert` runs in strict mode. The rules below are the authoritative reference. If you change them, update the skill body in `skills/opencode-agent-expert/SKILL.md` to match.

## Skills (`skills/<name>/SKILL.md`)

- `name` must match `^[a-z0-9]+(-[a-z0-9]+)*$` and be ≤64 chars.
- `name` must equal the directory name.
- `description` must be 1-1024 chars.
- `description` must contain "USE FOR" and "DO NOT USE FOR" cues.
- Frontmatter may only use `name`, `description`, `license`,
  `compatibility`, `metadata`. Everything else is ignored.

## Agents (`agents/<name>.md`)

- `description` is required and ≤300 chars.
- `mode`, if set, must be one of `primary`, `subagent`, `all`.
- Filename (without `.md`) becomes the agent name and must be unique.
- Do **not** shadow built-ins (`build`, `plan`, `general`, `explore`,
  `scout`) without explicit justification in the PR description.
- No deprecated `maxSteps` — use `steps`.
- No deprecated `tools` block when `permission` would do.
- `permission.task` must never include `"*": "allow"` and must deny
  the agent's own name to prevent self-invocation loops.
- Read-only agents must have `bash: deny` (or a strict glob map).
- **All agents except `scout` must include PDF access rule** (enforced):
  `read: { "anonymised*": "allow", "*.pdf": "ask" }`. Rationale: Protect
  against unintended processing of sensitive local PDF files. `scout` is
  exempt (internet-focused, not local filesystems).

## Commands (`commands/<name>.md`)

- `description` is required.
- `agent`, if set, must refer to an existing agent.
- `subtask: true` is the default for any command that runs a
  specialised workflow; only set it to `false` if you want the
  command to run in the primary context.
- Use the right placeholder syntax: `$ARGUMENTS`, `$1..$N`,
  `` !`shell` ``, `@file`.

## Cross-cutting

- One artifact per file. No multi-agent mega-files.
- Commit messages: short, imperative, one-line summary plus
  optional body. Example: `agents: add code-reviewer subagent`.
- License: MIT (see `LICENSE`).

---
> Source: [boscorat/opencode-config](https://github.com/boscorat/opencode-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
