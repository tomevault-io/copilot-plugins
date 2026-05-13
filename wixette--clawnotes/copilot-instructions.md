## clawnotes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo (`clawnotes`) contains tech notes and source snapshots of the **OpenClaw** codebase (an open-source personal AI assistant). Snapshots live under `openclaw-snapshots/<date>/`. Tech notes live under `docs/`. This is a read-only reference — not the live repo. Use it to study architecture, conventions, and patterns.

## Repository Layout

```
docs/                           # Tech notes (organized by subsystem)
├── README.md                   # Codebase overview (entry point)
├── agents/                     # Agent subsystem (7 docs)
├── channels/                   # Channel integrations (20+ docs)
├── gateway-*.md                # Gateway architecture + internals
├── cli-ui.md, plugins.md, ...  # Other subsystem docs
openclaw-snapshots/
└── 20260312/                   # Source snapshot (read-only reference)
    ├── src/                    # Core TypeScript (ESM)
    ├── extensions/             # ~40 channel/plugin workspace packages
    ├── apps/                   # iOS, Android, macOS companion apps
    ├── Swabble/                # Wake-word daemon (Swift 6.2)
    └── ...
```

## OpenClaw Architecture (Big Picture)

- **Gateway**: The core runtime — a Node.js process that orchestrates AI model calls, tool execution, and message routing across channels. Runs as a menubar app on macOS or a CLI daemon elsewhere.
- **Channels**: Messaging integrations (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, IRC, MS Teams, Matrix, etc.). Core channels live in `src/`, extensions in `extensions/`. Refactors to shared logic (routing, allowlists, pairing) must consider ALL channels.
- **Agents/Subagents**: AI agent harness with tools, skills, cron, and hooks. Skills publish to ClawHub, not core.
- **Companion Apps**: iOS (Swift/SwiftUI), Android (Kotlin), macOS (Swift menubar app) connect to the gateway via mDNS discovery + mutual TLS.
- **Swabble**: Standalone Swift CLI daemon for on-device wake-word detection (Speech.framework). Produces `SwabbleCore` (speech pipeline) and `SwabbleKit` (wake-word gate utilities).
- **Plugins**: npm packages loaded at runtime. Plugin-only deps stay in extension `package.json`, not root. MCP support via external `mcporter` bridge.
- **Canvas (A2UI)**: Live rendered UI surface controlled by the assistant.

## Key Development Commands (OpenClaw Core)

| Task | Command |
|------|---------|
| Install deps | `pnpm install` |
| Build/typecheck | `pnpm build` |
| TypeScript check | `pnpm tsgo` |
| Lint + format | `pnpm check` |
| Format fix | `pnpm format:fix` |
| Tests | `pnpm test` (vitest) |
| Coverage | `pnpm test:coverage` |
| Run CLI (dev) | `pnpm openclaw ...` or `pnpm dev` |
| Single test | `pnpm test <path>` |
| Low-memory tests | `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test` |

Runtime: Node 22+. Bun supported for TS execution. Prefer `pnpm` for builds.

## Key Development Commands (Swabble)

| Task | Command |
|------|---------|
| Build | `swift build` |
| Test | `swift test` |
| Format | `./scripts/format.sh` |
| Lint | `./scripts/lint.sh` |

Swift 6.2, swift-testing framework. Targets macOS 15+/iOS 17+.

## Coding Conventions

- **TypeScript**: ESM, strict typing, no `any`. Oxlint + Oxfmt for linting/formatting.
- **Swift (iOS/macOS)**: Prefer `Observation` framework (`@Observable`) over `ObservableObject`/`@StateObject`.
- **Tests**: Colocated `*.test.ts`, Vitest with 70% V8 coverage thresholds.
- **Commits**: Use `scripts/committer "<msg>" <files>` to keep staging scoped. Action-oriented messages (e.g., `CLI: add verbose flag`).
- **File size**: Aim for <500-700 LOC; split when it improves clarity.
- **No prototype mutation**: Use explicit inheritance/composition.
- **No `@ts-nocheck`** or disabled `no-explicit-any`.
- **Dynamic imports**: Don't mix `await import("x")` and static `import ... from "x"` for the same module. Use `*.runtime.ts` boundaries for lazy loading.
- **Tool schemas**: No `Type.Union`/`anyOf`/`oneOf`; use `stringEnum`/`Type.Optional`.
- **Naming**: "OpenClaw" in prose/headings; `openclaw` for CLI/package/paths.

## Important Conventions from AGENTS.md

- File references in chat must be repo-root relative (e.g., `extensions/bluebubbles/src/channel.ts:80`).
- Patched dependencies (`pnpm.patchedDependencies`) must use exact versions.
- Never update the Carbon dependency.
- Control UI uses Lit with legacy decorators (`@state()`, `@property()`), not standard decorators.
- Version locations span `package.json`, Android `build.gradle.kts`, iOS/macOS `Info.plist` files, and docs.
- Changelog: user-facing changes only, append to end of section, one contributor mention per line.

---
> Source: [wixette/clawnotes](https://github.com/wixette/clawnotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
