## runcodingplan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**WhichCC** (`runcodingplan`) — NPX CLI to switch Claude Code between alternative AI providers (ZAI, Kimi, MiniMax, Alibaba, and custom). The CLI generates per-session JSON settings files and spawns Claude Code with them.

- **Zero runtime dependencies** — only Node.js built-ins
- **ESM-only** — `"type": "module"` in package.json
- **Strict TypeScript** — `strict: true`, `noUncheckedIndexedAccess: true`
- **Node.js 18+** — uses built-in `fetch`, no polyfills
- **Packaged by tsup** as single binary with shebang

## Common Commands

```bash
npm run build       # tsup → dist/index.js
npm run dev         # tsup --watch (development)
npm run typecheck   # tsc --noEmit
npm run test        # vitest run
npm run test:watch  # vitest (watch mode)
npx .               # Run built CLI from project root (test shebang)
```

Pre-publish check: `npm run typecheck && npm run test && npm run build`.

## Architecture

### Entry Point Routing (`src/index.ts`)

The CLI routes based on flags. Priority order matters:

```
help → version → update → list → status → clean → showTemplate → resetTemplate
→ addCustom → removeCustom
→ provider + addModel → provider + removeModel → provider + removeKey
→ provider + model + setDefault + no apikey → setDefaultModel
→ provider + apikey + no model → setApiKey
→ provider (any) → launchCommand
→ no args → interactive
```

When both `provider` and `apikey` are given without a model, it **sets the key** rather than launching.

### Provider Resolution (`src/core/providers.ts`)

`resolveProvider(id, config)` checks built-in registry first, then custom providers. For built-in providers, it **merges** registry models with `config.providers[id].userModels[]` — registry models first, then userModels not already present. Custom providers use their `models[]` field directly (no registry merge).

`resolveAllProviders` returns all built-in + all custom as `ResolvedProvider[]`.

### Session Generation (`src/core/session.ts` + `src/core/template.ts`)

Session JSON is built from a **template file** at `~/.claude/.runcodingplan/template.json`. The template uses `[[VARIABLE]]` placeholders replaced at runtime via `applyTemplate()`. If no custom template exists, `DEFAULT_TEMPLATE_OBJECT` is used — identical structure to what the project plan describes.

Key: the template is user-editable (`--show-template`, `--reset-template`), allowing customization of the generated session JSON shape.

### API Key Encryption (`src/core/keys.ts`)

AES-256-GCM encryption. Key derived from `hostname() + username() + "runcodingplan-salt"` via PBKDF2 (100k iterations). This means `keys.json` is **machine-specific** — copying it to another machine will fail to decrypt.

API keys are stored as `"enc:v1:aes256gcm:<base64>"`.

### File Locations

All state lives under `~/.claude/.runcodingplan/`:

| File | Purpose |
|------|---------|
| `config.json` | WhichCC config: defaults, per-provider userModels, custom providers, lastUsed |
| `keys.json` | Encrypted API keys (one per provider, or `null`) |
| `registry.json` | Cached remote model registry (fetched via `-u`) |
| `template.json` | User-customizable session template |

Session files are written to `~/.claude/runcodingplan-{provider}-{timestamp}.json` (not into `.runcodingplan/`). Session files older than 24h are auto-cleaned on launch.

### Built-in Registry (`src/core/registry.ts`)

`BUILTIN_PROVIDERS` is the hardcoded source of truth for the 4 built-in providers (zai, kimi, minimax, alibaba). `getBuiltinProvider(id)` and `getAllBuiltinProviders()` expose it. Custom providers are stored in `config.json.customProviders`, not here.

### Config Structure (`src/core/config.ts`)

`loadConfig()` creates the default config file on first run. `migrateConfig()` handles upgrading old config versions. Default settings (`skipDangerous: false`, `statusLine: true`, `statusLineCommand: 'npx -y ccstatusline@latest'`) come from `src/constants.ts`.

### CLI Argument Parser (`src/cli/parser.ts`)

Zero-dependency parser. Throws `ParseError` on unknown flags or missing values. Flags with optional values (like `-sd`) are `boolean`; flags that need a value throw if the next arg looks like a flag.

### Interactive Mode (`src/cli/interactive.ts` + `src/cli/commands/interactive-flow.ts`)

Uses `process.stdin.setRawMode(true)` with keypress handling. Arrow keys navigate, Enter confirms. Always restores terminal on exit via `restoreTerminal()`. Components: `Select` (arrow nav + sections + separators), `TextInput` (masked or plaintext), `Confirm` (Y/n).

## Important Constraints

- Never touch `~/.claude/settings.json` — only create session files
- Session files use prefix `runcodingplan-` (not `whichcc-`) in `~/.claude/`
- Custom provider IDs cannot use reserved names: `zai`, `kimi`, `minimax`, `alibaba`
- Registry update (`-u`) auto-promotes user-added models that now appear in the official registry (removes `(*)` marker)
- `RUNCP_HOME_DIR` env var overrides `~/.claude` (for testing)
- `RUNCP_CLAUDE_BIN` env var overrides Claude binary resolution

---
> Source: [ersinkoc/RunCodingPlan](https://github.com/ersinkoc/RunCodingPlan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
