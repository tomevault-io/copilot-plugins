## agent-notion-template-docs

> notion-doc is an agent skill that renders documents with Notion's blocks and

# AGENTS.md

notion-doc is an agent skill that renders documents with Notion's blocks and
design tokens. It ships as a portable Agent Plugin (Codex, ChatGPT) and as a
Claude Code plugin from the same files.

## Repo layout that matters

```text
plugin.json              # portable manifest (Agent Plugins 1.0) — canonical
.codex-plugin/           # Codex compatibility manifest
.claude-plugin/          # Claude Code manifest + marketplace catalog
.agents/plugins/         # Codex marketplace catalog
.agents/skills/notion-doc -> ../../skills/notion-doc   # symlink, not a copy
hooks.json               # Codex PostToolUse wiring
hooks/hooks.json         # Claude Code PostToolUse wiring
skills/notion-doc/       # the skill itself — SKILL.md, template.html, lint.py
```

Both runtimes load the same `skills/` and the same hook script. Never fork the
skill per agent: `.agents/skills/notion-doc` is a symlink on purpose.

## Rules

- **`template.html`'s `<style>` block is frozen.** Everything downstream — the
  linter, the hook, the examples — compares against it byte for byte. Changing
  it is a deliberate canon change, and every file under `examples/` has to be
  regenerated in the same commit.
- **Five manifests carry the version string** (`plugin.json`,
  `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, and both
  marketplace catalogs). Bump them together; CI fails if they disagree.
- **The READMEs are a pair.** `README.md` and `README.ko.md` document the same
  thing in two languages — change both or neither.
- **The hook must never block work.** `hooks/notion-doc-lint.py` fails open:
  any exception prints `{}` and exits 0. Keep it that way.
- Hook payloads differ per runtime (Claude sends `tool_input.file_path`, Codex
  sends an `apply_patch` body). Parse both; `hooks/test_gating.py` pins it.
- Standard library only, in the skill and in the hook. It has to run wherever
  the agent runs.

## Checks before you finish

```bash
python3 skills/notion-doc/lint.py skills/notion-doc/template.html examples/*.html
python3 hooks/test_gating.py
```

Both are what CI runs (`.github/workflows/lint.yml`). The weekly
`notion-sync` job needs network access and is not expected to pass locally.

## Writing documents in this repo

The skill applies to itself. If you generate an HTML document here, start from
`skills/notion-doc/template.html` and follow `skills/notion-doc/SKILL.md` — do
not write new CSS.

---
> Source: [heyman333/agent-notion-template-docs](https://github.com/heyman333/agent-notion-template-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
