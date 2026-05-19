## skills

> Each skill directory contains a `SKILL.md` and optional supporting files. Skills may live at the repo root or under collection directories such as `skills-stable/` and `skills-beta/`. `skillshare` syncs skills from its configured source directory to AI tool config directories.

# Guidelines for AI Agents

## Repository layout

Each skill directory contains a `SKILL.md` and optional supporting files. Skills may live at the repo root or under collection directories such as `skills-stable/` and `skills-beta/`. `skillshare` syncs skills from its configured source directory to AI tool config directories.

Directories prefixed with `_` are externally synced, gitignored, and overwritten on update. Never edit them.

Non-`_` directories listed in `.metadata.json` are also externally synced. Do not patch them for local-only workflows; `skillshare update` may overwrite the edits. Put local extensions in a first-party companion skill instead, or upstream the change.

```text
<skillshare source>/
├── <skill-name>/        # A root-level skill
├── skills-stable/       # Stable skill collection
├── skills-beta/         # Beta skill collection
├── _<org>-skills/       # Org skills (gitignored)
└── _<community>/        # Community skills (gitignored)
```

## Required workflow

- `prek` must be configured before committing. If hooks are not installed, run `prek install` to install both `pre-commit` and `pre-push` hooks.
- Never commit directly to `main`. Create a branch and open a PR for changes that should be merged.

## Code style

For first-party files you edit: 4-space indent everywhere, 2-space for Markdown. LF line endings and final newlines are required.

- **Python** — Ruff follows `pyproject.toml` (selects E/W/F/UP/B/SIM/I/TID plus project-specific extend/ignore rules, line-length 120, target py314). Format: `uv run ruff format --check .`. Lint: `uv run ruff check .`. Type-check: `uv run ty check .`.
- **JS / JSON** — Biome (double quotes, 4-space indent). Lint: `biome ci .`.
- **TOML** — `tombi` formatter, 4-space indent.
- **Markdown** — `markdownlint-cli2`.
- **Spelling** — `typos`.

## Skillshare operations

Always use non-interactive flags (`--force`, `--all`, `--yes`). AI agents cannot answer prompts. Before running `skillshare sync`, verify `skillshare status --json` reports this repository as `source.path`. Always run `skillshare sync` after any mutation (`install`, `uninstall`, `update`, `collect`, `target`). Use `--json` when you need to parse output.

## External skill lint excludes

Non-`_` externally synced skills are tracked in `.metadata.json` and must be excluded from all lint configs. Find them with:

```bash
jq -r '.entries | keys[] | select(startswith("_") | not)' .metadata.json | sort
```

Add each directory to these six config files (eight places total — `pyproject.toml` has three sections):

- `.typos.toml` — `[files].extend-exclude`
- `.markdownlint-cli2.yaml` — `ignores`
- `biome.jsonc` — `files.includes` with `!!dir` force-ignore (no trailing `/`)
- `pyproject.toml` — `[tool.ruff].exclude` and `[tool.ty.src].exclude` use trailing-slash directory paths (`dir/`); `[tool.tombi.files].exclude` uses `dir/**` globs
- `prek.toml` — top-level `exclude` regex
- `.autocorrectignore`

`_`-prefixed directories are gitignored and never checked in, so they don't need lint excludes. Only non-`_` directories listed in `.metadata.json` need them.

When deleting a skill, remove its directory **and** remove its entries from all six config files listed above. Use `skillshare uninstall` when possible; if you `rm -rf` manually, you must clean the configs yourself.

After a skillshare version upgrade or `skillshare update`, directory names may change or entries may disappear. Always verify that all six config files still match `.metadata.json` — the exclude lists must be an exact 1:1 match with the non-`_` entries.

Run `mise run check-managed-skill-integrity` before committing. It uses `skillshare audit --analyzer integrity` to ensure every non-`_` `.metadata.json` entry still matches its pinned file hashes.

---
> Source: [jihuanshe/skills](https://github.com/jihuanshe/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
