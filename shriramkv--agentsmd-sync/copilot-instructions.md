## agentsmd-sync

> `agentsmd-sync` propagates a canonical AGENTS.md into the tool-specific

# AGENTS.md

## Project

`agentsmd-sync` propagates a canonical AGENTS.md into the tool-specific
instruction files that other assistants read (CLAUDE.md, Copilot, Cursor,
Windsurf, Cline, Gemini), and gates CI when they drift apart. It is the
propagation step that completes the AGENTS.md quartet: `agentsmd-init`
creates, `agentsmd-lint` validates one file, `agentsmd-coverage` measures
presence, `agentsmd-resolve` computes the effective doc, and this syncs it
outward.

## Constraints

- Stdlib-only. Do not add dependencies.
- Target Python 3.9+.
- Generated content always lives inside the managed-block markers. Never
  write outside them, and always preserve hand-written content that sits
  outside the block.
- Changing the marker strings in targets.py breaks every already-synced
  repo. Treat marker format as a stable contract.

## Layout

- `agentsmd_sync/targets.py`  target fleet + managed-block markers
- `agentsmd_sync/sync.py`     evaluate / apply / drift logic
- `agentsmd_sync/cli.py`      argparse CLI and the check gate
- `tests/`                    unittest suite
- `samples/`                  example AGENTS.md

## Testing

Run `python -m unittest discover -s tests -v` before committing. CI runs the
same suite on the 3.9-3.12 matrix. Every new behaviour needs a test.

---
> Source: [shriramkv/agentsmd-sync](https://github.com/shriramkv/agentsmd-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
