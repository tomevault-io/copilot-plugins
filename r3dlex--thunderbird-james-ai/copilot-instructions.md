## thunderbird-james-ai

> Thunderbird WebExtension (Manifest V3) bringing multi-provider AI directly into the mail client. No external backend. Pure TypeScript, Angular, and the Thunderbird MailExtension API.

# Corvus — AI-Powered Thunderbird Extension

Thunderbird WebExtension (Manifest V3) bringing multi-provider AI directly into the mail client. No external backend. Pure TypeScript, Angular, and the Thunderbird MailExtension API.

## Quick Reference

- **Target**: Thunderbird 148+, Manifest V3
- **Language**: TypeScript (strict mode), Python 3.12+ (tooling only)
- **UI**: Angular 19+ with AOT compilation
- **Build**: `npm run build` (webpack + Angular CLI)
- **Test**: `npm test` (Jest for background), `npm run test:ui` (Angular)
- **Package**: `npm run package` (produces corvus.xpi)
- **Lint**: `npm run lint`
- **Pipelines**: `cd tools/pipeline_runner && poetry run pipeline-runner <name>`

## Hard Constraints

- No external backend services - everything runs inside the extension
- No localStorage/sessionStorage - use `messenger.storage.local`
- No JIT compilation, no eval(), no Function() constructors
- No emojis, no semicolons in user-facing strings
- API keys stored encrypted via AES-GCM in messenger.storage.local
- All AI provider communication via raw fetch() - no SDK dependencies

## Agent Guidelines

See [AGENTS.md](AGENTS.md) for agent role definitions and task delegation patterns.

## Specifications

Detailed specs live in `spec/` using progressive disclosure:

| File | Contents |
|------|----------|
| [spec/architecture.md](spec/architecture.md) | System architecture, directory structure |
| [spec/ai-providers.md](spec/ai-providers.md) | Provider abstraction, API details per provider |
| [spec/tools.md](spec/tools.md) | AI tool/function calling definitions |
| [spec/features.md](spec/features.md) | Feature specs: chat, assistant, compose, rules engine |
| [spec/ui.md](spec/ui.md) | UI layout, theme, component structure |
| [spec/security.md](spec/security.md) | Encryption, privacy, API key management |
| [spec/testing.md](spec/testing.md) | Testing strategy, coverage targets |
| [spec/build.md](spec/build.md) | Build pipeline, webpack, Angular CLI |

## Architecture Decisions

ADRs in `docs/adr/`, managed via archgate. Run `npx archgate list --dir docs/adr` to view.

## Key Directories

```
src/background/     # Background script (AI, storage, rules, tools)
src/ui/             # Angular UI (popup panels)
src/types/          # Thunderbird API type declarations
tools/              # Python pipeline runner for CI/CD
docs/adr/           # Architecture Decision Records
spec/               # Detailed specifications
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r3dlex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
