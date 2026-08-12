## embedded-engineering-skill

> `SKILL.md` is the main entrypoint. Python tools in `scripts/` by domain (eda/, rf/, digital/, protocol/, system/). Templates in `circuits/templates/`, parts in `components/library.json`, refs in `references/`, subagent prompts in `subagents/`.

# Repository Guidelines

## Structure
`SKILL.md` is the main entrypoint. Python tools in `scripts/` by domain (eda/, rf/, digital/, protocol/, system/). Templates in `circuits/templates/`, parts in `components/library.json`, refs in `references/`, subagent prompts in `subagents/`.

## Commands
Python: `C:\Users\24560\AppData\Local\Programs\Python\Python312\python.exe -X utf8`

```powershell
# Compile check
python -m py_compile scripts\eda\gen_kicad_project.py scripts\eda\gen_template_gallery.py scripts\eda\gen_jlc_package.py scripts\eda\validate_eda_outputs.py scripts\eda\validate_project_spec.py scripts\eda\gen_easyeda_std.py scripts\eda\render_design_preview.py scripts\eda\erc_check.py

# Gallery regression
python -X utf8 scripts\eda\gen_template_gallery.py --out C:\Users\24560\.codex\.tmp\embedded-engineering-gallery

# JLC package test
python -X utf8 scripts\eda\gen_jlc_package.py --manifest <path>\project.netlist.json

# Whitespace
git diff --check
```

## Style
4-space indent, type hints, snake_case filenames, UPPER_SNAKE_CASE constants, hyphenated template names (e.g. `esp32-c3-sensor-node.json`). Use `-X utf8` for Chinese/engineering symbols.

## Testing
No pytest — verification is script-driven. After eda/ changes: `py_compile` + regenerate gallery. After template changes: rerun affected generator, confirm outputs produced.

## Commits
Short imperative subjects. Body explains why for non-trivial changes. PRs list scope, paths, verification commands, and output paths for generated artifacts.

## Security
Don't commit generated output dirs, local machine paths beyond examples, or MCP secrets. `components/library.json` and templates are source-of-truth — update carefully.

---
> Source: [2456018331lby-dev/embedded-engineering-skill](https://github.com/2456018331lby-dev/embedded-engineering-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
