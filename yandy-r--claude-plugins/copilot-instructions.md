## claude-plugins

> This repository ships a single Claude Code plugin called `ycc`, distributed via the

# Claude Plugins - Project Instructions

## Overview

This repository ships a single Claude Code plugin called `ycc`, distributed via the
marketplace at `.claude-plugin/marketplace.json`. All skills, commands, and agents live
under `ycc/` and are accessed at runtime as `ycc:{skill}`, `/ycc:{command}`, or
`subagent_type: "ycc:{agent}"`.

The same source trees also generate native compatibility bundles for Cursor, Codex, and
opencode:

- Cursor bundle: `.cursor-plugin/`
- Codex bundle: `.codex-plugin/ycc/`
- Codex custom agents: `.codex-plugin/agents/`
- opencode bundle: `.opencode-plugin/` (skills, agents, commands, AGENTS.md, opencode.json)

> Pre-2.0 versions shipped 9 separate plugins. 2.0.0 collapsed them into one bundle so
> every skill is reachable via the same `ycc:` namespace prefix. See `docs/plans/` for
> the consolidation plan.

## Repository Layout

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json     # single ycc entry
├── .codex-plugin/
│   ├── agents/              # generated Codex custom agents
│   └── ycc/                 # generated Codex plugin root
├── .cursor-plugin/          # generated Cursor bundle
├── .opencode-plugin/        # generated opencode bundle
│   ├── skills/              # → ~/.config/opencode/skills/
│   ├── agents/              # → ~/.config/opencode/agents/
│   ├── commands/            # → ~/.config/opencode/commands/
│   ├── AGENTS.md            # opencode rules file
│   └── opencode.json        # schema + default model + MCP translation
├── ycc/
│   ├── .claude-plugin/
│   │   └── plugin.json      # name: "ycc", version: 2.0.0
│   ├── commands/            # slash commands (one .md per command)
│   ├── agents/              # agents (one .md per agent)
│   └── skills/
│       ├── _shared/         # cross-skill helper scripts
│       └── {skill-name}/
│           ├── SKILL.md     # skill prompt (required)
│           ├── references/  # templates, examples, reference docs
│           └── scripts/     # validation and helper scripts
└── docs/
    └── plans/               # implementation plans
```

## Plugin Development Conventions

### Naming

- The plugin name is `ycc` and must NOT change. The marketplace prefix is always `ycc:`.
- Skills: `kebab-case` directory under `ycc/skills/`. The directory name becomes the
  skill identifier (e.g., `ycc/skills/git-workflow/` → `ycc:git-workflow`).
- Commands: `kebab-case.md` under `ycc/commands/`. The basename becomes the slash
  command (e.g., `ycc/commands/clean.md` → `/ycc:clean`).
- Agents: `kebab-case.md` under `ycc/agents/`. The basename becomes the agent identifier
  (e.g., `ycc/agents/codebase-advisor.md` → `subagent_type: "ycc:codebase-advisor"`).
- Scripts: `kebab-case.sh` with bash shebang.

### Scripts

All scripts must:

- Start with `#!/usr/bin/env bash`
- Use `set -euo pipefail` for safety
- Include validation guards (check required inputs exist)
- Exit with meaningful codes (0 = success, 1 = error)
- Write output to stdout, errors to stderr

When a script needs to reference its own plugin path, use `${CLAUDE_PLUGIN_ROOT}` —
this resolves to the `ycc/` directory at runtime. Paths inside skills follow the form
`${CLAUDE_PLUGIN_ROOT}/skills/{skill-name}/...`.

Codex-generated skills are not source-edited directly. The Codex generator rewrites
those paths to the managed install location `~/.codex/plugins/ycc/...`.

### Skills

Each skill directory contains:

- `SKILL.md` — the skill prompt (required)
- `references/` — templates, examples, and reference docs
- `scripts/` — validation and helper scripts

### Cross-skill helpers

Shared helpers (used by more than one skill) live under `ycc/skills/_shared/scripts/`.
Skills source them via `${CLAUDE_PLUGIN_ROOT}/skills/_shared/scripts/{name}.sh`.

### Registration

The marketplace registry at `.claude-plugin/marketplace.json` contains a single entry:

```json
{
  "name": "ycc",
  "version": "2.0.0",
  "source": "./ycc"
}
```

Do not add additional plugin entries. New functionality goes into the existing `ycc`
plugin as a new skill, command, or agent. See `CONTRIBUTING.md` → Scope & Guardrails for
the full policy on what belongs in `ycc` and what to reject.

## Generated Compatibility Targets

- `ycc/skills/` is the source of truth for Cursor, Codex, and opencode skill generation.
- `ycc/agents/` is the source of truth for Cursor, Codex, and opencode agent generation.
- `ycc/commands/` is the source of truth for opencode command generation (Cursor does not
  natively consume `.md` commands; Codex has no slash-command layer).
- Do not hand-edit generated files under `.cursor-plugin/`, `.codex-plugin/`, or
  `.opencode-plugin/` unless you are first changing the generator.
- Codex does not support this repo's custom slash-command layer as installable artifacts.
  The native Codex target exposes skills via the plugin bundle and agents via TOML files.
- opencode has first-class support for skills, agents, AND commands. The opencode bundle
  ships all three, plus an `opencode.json` config (with MCP translated from
  `mcp-configs/mcp.json`) and an `AGENTS.md` rules file derived from this document.
- See [`ycc/skills/_shared/references/target-capability-matrix.md`](ycc/skills/_shared/references/target-capability-matrix.md)
  for the authoritative per-target capability table.

## Testing Changes

After modifying anything under `ycc/`:

1. Validate JSON with `python3 -m json.tool`:
   - `python3 -m json.tool .claude-plugin/marketplace.json`
   - `python3 -m json.tool ycc/.claude-plugin/plugin.json`
2. Verify all `${CLAUDE_PLUGIN_ROOT}` paths resolve (no broken references).
3. Confirm shell scripts remain executable:
   `find ycc/skills -name "*.sh" -not -executable` (should output nothing).
4. Test the skill or command in a live Claude Code session via its `ycc:` prefix.

If you changed `ycc/skills/`, `ycc/agents/`, or `ycc/commands/`, also regenerate and
validate the compatibility bundles. The recommended pair is the unified entrypoints:

```bash
./scripts/sync.sh         # regenerate inventory + cursor + codex + opencode bundles
./scripts/validate.sh     # run every validator (this is what CI runs)
```

Both accept `--only <targets>` with comma-separated values (`inventory, cursor, codex,
opencode, json`). The individual generator / validator scripts are still available
for targeted iteration:

1. Codex:
   - `./scripts/generate-codex-skills.sh`
   - `./scripts/generate-codex-agents.sh`
   - `./scripts/generate-codex-plugin.sh`
   - `./scripts/validate-codex-skills.sh`
   - `./scripts/validate-codex-agents.sh`
   - `./scripts/validate-codex-plugin.sh`
2. Cursor:
   - `./scripts/generate-cursor-skills.sh`
   - `./scripts/generate-cursor-agents.sh`
   - `./scripts/generate-cursor-rules.sh`
   - `./scripts/validate-cursor-skills.sh`
   - `./scripts/validate-cursor-agents.sh`
   - `./scripts/validate-cursor-rules.sh`
3. opencode:
   - `./scripts/generate-opencode-skills.sh`
   - `./scripts/generate-opencode-agents.sh`
   - `./scripts/generate-opencode-commands.sh`
   - `./scripts/generate-opencode-plugin.sh`
   - `./scripts/validate-opencode-skills.sh`
   - `./scripts/validate-opencode-agents.sh`
   - `./scripts/validate-opencode-commands.sh`
   - `./scripts/validate-opencode-plugin.sh`

## Precedence

1. System, developer, and explicit user instructions for the task.
2. This file and [`AGENTS.md`](AGENTS.md) as repo policy.
3. General best practices when nothing above conflicts.

## MUST / MUST NOT

- **Secrets**: **Never** commit `.env`, `.env.encrypted`, tokens, or API keys.
- **Issues**: Use the YAML form templates under `.github/ISSUE_TEMPLATE/` when present. **Do not** create title-only or template-bypass issues. If `gh issue create --template` fails, create the issue via GitHub API/tooling with a body that mirrors the form fields, then apply correct labels — **not** a vague one-liner.
- **Pull requests**: Follow `.github/pull_request_template.md` when present. **Always** link the related issue (`Closes #…`). **Label** PRs using the project taxonomy — **never** invent ad-hoc labels.
- **Commits**: Use **Conventional Commits 1.0.0** — `feat|fix|docs|refactor|perf|test|build|ci|chore(scope): …`. Write the title as you want it to appear in `CHANGELOG.md`.
- **Internal docs commits**: Files under `docs/plans`, `docs/research`, or `docs/internal` **must** use `docs(internal): …`. Other non-user-facing churn: prefer `chore(…): …` to stay out of release notes.
- **Large features**: Split into smaller phases and tasks with clear dependencies and order of execution.
- **File size (~500 lines)**: Aim for **around 500 lines** per file as a soft cap. Files that drift meaningfully past that **must** be refactored into smaller modules unless the content is inherently contiguous (generated code, schemas, large test fixtures). The intent is maintainability, not a hard ceiling.
- **Modularity & reuse**: Code **must** decompose into small, cohesive units — submodules, libraries, or reusable components — with a clear public surface and minimal cross-module coupling. **No copy-paste duplication** (DRY): extract shared logic into a shared module. Prefer composition over inheritance. Avoid circular dependencies.
- **Single responsibility**: Each function, module, and component **must** have one clear reason to exist. Split when a unit grows more than one responsibility.
- **MCP**: When an MCP server fits the task (GitHub, docs, browser, etc.), **prefer it**. **Read** each tool's schema/descriptor before calling.

## SHOULD (implementation)

### General

- **Naming**: Intention-revealing names for functions, types, and modules. Public APIs should read like documentation.
- **No dead code**: Remove unused code, imports, and commented-out blocks. Git preserves history.
- **Dependency hygiene**: Before adding a new dependency, check whether an existing one does the job. New deps need a justification (maintenance cost, license, security).
- **Fail fast at boundaries**: Validate inputs at module and system boundaries; propagate via typed errors. Never silently swallow errors.
- **Tests alongside changes**: New or modified behavior ships with tests in the same change.

### Languages

- **Python** (`scripts/generate_*.py`): PEP 8 throughout; type hints required on all public API signatures; prefer `ruff` for linting and `mypy --strict` for type checking.
- **Shell** (`scripts/*.sh`, `ycc/skills/*/scripts/*.sh`): `#!/usr/bin/env bash` + `set -euo pipefail`; validation guards on required inputs; stdout for results, stderr for errors; exit 0 on success, 1 on error.

## Git & Conventional Commits

This project uses **Conventional Commits 1.0.0**. Every commit title must match:

```
<type>[optional scope]: <description>
```

### Types

| Type       | Purpose                                     | Version bump |
| ---------- | ------------------------------------------- | ------------ |
| `feat`     | New user-facing feature                     | minor        |
| `fix`      | User-facing bug fix                         | patch        |
| `docs`     | Documentation only                          | —            |
| `refactor` | Code change that is neither fix nor feature | —            |
| `perf`     | Performance improvement                     | —            |
| `test`     | Adding or correcting tests                  | —            |
| `build`    | Build system or external dependency changes | —            |
| `ci`       | CI/CD configuration changes                 | —            |
| `chore`    | Other non-user-facing changes               | —            |
| `style`    | Formatting/whitespace only                  | —            |

### Scope

`feat(auth): …` — scope is the module, crate, package, or area of change. Keep it concise.

### Breaking changes

Append `!` after the type/scope (`feat!: …`) **or** add a `BREAKING CHANGE: …` footer. Either triggers a major version bump.

### Internal docs

Use `docs(internal): …` for files under `docs/plans`, `docs/research`, or `docs/internal`. These stay out of release notes.

## GitHub Workflow

- **Labels**: Use only the project's defined label taxonomy (`type:`, `area:`, `priority:`, `status:` families). Never create ad-hoc labels.
- **Issues**: File an issue before starting non-trivial work. Link the issue number in the PR (`Closes #…`).
- **PRs**: Follow the PR template; fill every checklist item honestly. Small, focused PRs over large omnibus ones.

## Stack Overview

| Layer            | Technology                   | Notes                                             |
| ---------------- | ---------------------------- | ------------------------------------------------- |
| Primary language | **mixed** (Shell + Markdown) | Plugin source under `ycc/`                        |
| Generators       | Python 3                     | `scripts/generate_*.py` emit Cursor/Codex bundles |
| Helper scripts   | Bash                         | `scripts/lint.sh`, `scripts/format.sh`            |
| Package manager  | npm                          | Drives lint/format only — no test or build target |

## Commands

```bash
# Lint (Python + Shell)
npm run lint              # or: scripts/lint.sh --python --shell
npm run lint:modified     # only files changed vs. git HEAD
npm run lint:fix          # auto-apply fixes

# Format
npm run format            # Python + docs
npm run format:modified   # only modified files
```

New projects should bootstrap this same lint/format environment via `/ycc:formatters` (or `/ycc:init --formatters`), which installs the `scripts/style.sh` bundle, tool configs, aliases, and docs into the target repo.

Testing and validation are defined in `## Testing Changes` above — JSON validation plus the Codex/Cursor generate-and-validate pipelines are the real verification loop for this repository.

---
> Source: [yandy-r/claude-plugins](https://github.com/yandy-r/claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
