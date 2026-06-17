## epub2skill

> Guide for how an agent should use the epub2skill CLI.


# Agent CLI Usage

Use this skill to run `epub2skill` correctly and consistently.

## Goal

Convert one or more EPUB files into an agent-usable skill directory with `SKILL.md`, `bookindex.md`, and `resources/`.

## Command Template

```bash
epub2skill "<epub-or-glob>" \
  --skill <skill-name> \
  --targets <target-list> \
  --description "<skill-description>"
```

## Required Rules

- Always set `--skill` using kebab-case.
- Always verify input path or glob matches `.epub` files.
- Prefer `--targets opencode-project` unless user asks otherwise.
- Add `--force` only when user explicitly wants overwrite behavior.

## Target Reference

- `claude-project`: `<project>/.claude/skills/<skill-name>`
- `claude-global`: `~/.claude/skills/<skill-name>`
- `opencode-project`: `<project>/.opencode/skills/<skill-name>`
- `opencode-global`: `~/.config/opencode/skills/<skill-name>`
- `qwen-project`: `<project>/.qwen/skills/<skill-name>`
- `qwen-global`: `~/.qwen/skills/<skill-name>`

## Recommended Workflow

1. Confirm EPUB source path or glob.
2. Choose a clear `--skill` name.
3. Choose target(s) based on user scope (project vs global).
4. Run conversion command.
5. Verify generated files:
   - `SKILL.md`
   - `bookindex.md`
   - `resources/**/*.md`

## Example Commands

```bash
epub2skill "./books/meditations.epub" \
  --skill stoicism-guide \
  --targets opencode-project \
  --description "Answer based on Meditations by Marcus Aurelius"
```

```bash
epub2skill "./books/*.epub" \
  --skill philosophy-library \
  --targets opencode-project,claude-project \
  --merge \
  --description "Ground responses in converted philosophy books"
```

## Output Validation

- If no EPUB matches, stop and report the unmatched path/glob.
- If target already exists and `--force` is not set, stop and report conflict.
- After success, report:
  - number of EPUB files converted
  - target directories written
  - whether merge mode was used

---
> Source: [yearzen1/epub2skill](https://github.com/yearzen1/epub2skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
