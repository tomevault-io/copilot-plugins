## pion

> A simple messaging bridge connecting WhatsApp/Telegram to pi-agent.

# Pion - Agent Instructions

A simple messaging bridge connecting WhatsApp/Telegram to pi-agent.

## Quick Context

**Read these first when starting a session:**
1. `docs/architecture.md` - System design, data layout, caching strategy
2. `docs/runtime.md` - Daemon/TUI architecture, systemd

## Philosophy: KISS + TDD

This project prioritizes simplicity over features. Before adding anything, ask:
- Is this essential for the core use case?
- Can this be a skill instead of core code?
- Will this make the codebase harder to understand?

**Target: <5k lines of code** for the core.

### Test-Driven Development

We follow TDD strictly:

1. **Write test first** - Define what the feature should do
2. **Run test, verify it fails** - Confirms test is actually testing something
3. **Implement the feature** - Minimal code to make test pass
4. **Run test, verify it passes** - Feature works as specified
5. **Refactor if needed** - Clean up while tests keep you safe

```bash
# TDD workflow
bun test --watch  # Keep this running

# 1. Write test in test/foo.test.ts
# 2. See it fail (red)
# 3. Write code in src/foo.ts
# 4. See it pass (green)
# 5. Refactor if needed
```

**No exceptions.** Even for "quick fixes" - write the test first.

## Architecture Overview

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│  Providers  │────▶│  Router  │────▶│   Runner    │
│  (WA/TG)    │     │          │     │  (pi-agent) │
└─────────────┘     └──────────┘     └─────────────┘
       │                 │                  │
       ▼                 ▼                  ▼
┌─────────────┐    ┌──────────┐     ┌─────────────┐
│  Commands   │    │  Config  │     │  Workspace  │
│  (/new etc) │    │  (yaml)  │     │  + Skills   │
└─────────────┘    └──────────┘     └─────────────┘
```

## Directory Structure

```
src/
├── providers/           # Platform adapters (Telegram, WhatsApp)
│   ├── telegram.ts
│   ├── telegram-format.ts
│   ├── telegram-tools.ts
│   ├── whatsapp.ts
│   └── types.ts
├── core/                # Router, Runner, Workspace, Skills, Commands, Compactor
│   ├── router.ts
│   ├── runner.ts
│   ├── workspace.ts
│   ├── skills.ts
│   ├── commands.ts
│   └── compactor.ts
├── config/              # Schema and loading
│   ├── schema.ts
│   └── loader.ts
├── tui/                 # Monitor TUI
│   └── monitor.ts
├── daemon.ts            # Main daemon entry point
└── index.ts             # Library exports

docs/
├── architecture.md      # System design
└── runtime.md           # Daemon/TUI architecture

test/                    # Mirrors src/ structure
```

## Stack

- **Runtime**: Bun
- **Telegram**: grammy
- **WhatsApp**: @whiskeysockets/baileys
- **Agent**: pi packages (pi-agent-core, pi-ai, pi-coding-agent, pi-tui)
- **Config**: YAML (`~/.pion/config.yaml` or `./pion.yaml` for dev)
- **Lint**: Biome

## Quick Commands

```bash
bun test              # Run all tests
bun test --watch      # Watch mode
bun run lint          # Biome check
bun run daemon        # Run daemon
bun run dev           # Watch mode (daemon)
bun run monitor       # Session monitor TUI
bun run whatsapp:pair # WhatsApp QR pairing
```

## Runtime

- Config: `~/.pion/config.yaml` (preferred) or `./pion.yaml` (dev fallback)
- Example config: `pion.example.yaml` at repo root
- Runtime dir: `~/.pion/` with subdirs: `sessions/`, `skills/`, `whatsapp-auth/`, `agents/main/`
- Auth: `~/.pion/auth.json` (Anthropic OAuth, shared with pi)

## Conventions

- Concise responses preferred
- Uses Bun, not npm/yarn
- pi-mono cloned at ~/Projects/pi-mono (for reference)

---
> Source: [ogulcancelik/pion](https://github.com/ogulcancelik/pion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
