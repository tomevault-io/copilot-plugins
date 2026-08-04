## hawk-hooks

> `hawk-hooks` is a multi-tool component manager with registry-backed sync.

# hawk-hooks

`hawk-hooks` is a multi-tool component manager with registry-backed sync.

## Primary Architecture

```
src/hawk_hooks/
├── cli.py                  # Main CLI entry (hawk + hawk-hooks commands)
├── config.py               # YAML config + directory index + package index
├── sync.py                 # Sync/clean orchestration + cache + result formatting
├── resolver.py             # Global/profile/dir-chain resolution
├── registry.py             # Registry add/remove/replace/list operations
├── downloader.py           # Git/local package scan + metadata-aware classification
├── event_mapping.py        # Canonical hook event contract + per-tool support
├── adapters/               # Claude, Gemini, Codex, OpenCode, Cursor, Antigravity
└── interactive/            # Dashboard, toggles, wizard, settings editor
```

## Config + Data Locations

- Global config: `~/.config/hawk-hooks/config.yaml`
- Registry: `~/.config/hawk-hooks/registry/`
- Profiles: `~/.config/hawk-hooks/profiles/*.yaml`
- Packages index: `~/.config/hawk-hooks/packages.yaml`
- Per-project config: `<project>/.hawk/config.yaml`
- Sync cache: `~/.config/hawk-hooks/cache/resolved/`

## Core Concepts

- Registry-managed components: `skills`, `hooks`, `prompts`, `agents`, `mcp`.
- Resolution chain: `global` -> registered parent dirs -> current project (with optional profiles).
- Per-tool adapters own concrete sync behavior and capability warnings.
- Hook events use a canonical contract in `event_mapping.py` with explicit unsupported handling.

## Typical Workflow

```bash
hawk init
hawk add <type> <path>
hawk sync
hawk status
```

For package-based workflows:

```bash
hawk download <url>
hawk packages
hawk update
```

---
> Source: [pkronstrom/hawk-hooks](https://github.com/pkronstrom/hawk-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
