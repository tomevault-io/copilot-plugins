## skills

> When editing this repository, preserve skill-tree integrity, relative links, and packaging layout.

# Zoom Skills Repository

This repository uses a packaging root plus an installable skill tree:
- packaging/docs at repo root
- installable skills under `skills/`

## Rules

- Keep skill content under `skills/`
- Keep packaging manifests at repo root:
  - `.claude-plugin/`
  - `.cursor-plugin/`
  - `.mcp.json`
- Use repository-relative markdown links only
- Do not commit machine-local absolute paths
- Keep `skills/SKILL.md` as the skill-bundle entrypoint

## After structural edits

Run:

```bash
python3 tools/md-audit/md_audit.py --root . --exclude raw-docs --exclude md-graph --exclude tools/zoom-crawler
node md-graph/build-graph.mjs . md-graph/data.json
```

Copy the refreshed graph to the hosted viewer when applicable.

---
> Source: [zoom/skills](https://github.com/zoom/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
