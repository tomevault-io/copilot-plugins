## openkilo

> Guidance for agentic coding assistants working in this repository.

# AGENTS.md

Guidance for agentic coding assistants working in this repository.

## Project Overview

- **Package**: `openkilo`
- **Purpose**: OpenCode plugin for accessing 40+ free AI models through Kilo Gateway
- **Language**: TypeScript (ESM, strict mode)
- **Runtime**: Node.js with Bun tooling

## Commands (Bun Preferred)

```bash
# Install dependencies
bun install

# Build
bun run build

# Type check
bun run typecheck

# Lint
bun run lint

# Format
bun run format
```

## Architecture

```
src/
├── index.ts          # Plugin entry point (config hook + tools)
├── registry.ts       # Kilo Gateway model fetcher
├── transform.ts      # Model schema transformer (Kilo → OpenCode)
├── auth/
│   ├── index.ts      # Auth exports
│   ├── api-key.ts    # API key authentication
│   ├── oauth.ts      # OAuth browser flow
│   └── storage.ts    # Credential persistence (~/.config/openkilo/auth.json)
└── tools/
    ├── index.ts      # Tool exports
    ├── login.ts      # kilo_login, kilo_logout, kilo_status
    └── refresh.ts    # kilo_refresh, kilo_models
```

## Key Concepts

### Kilo Gateway

- **URL**: `https://api.kilo.ai/api/openrouter/`
- **NPM**: `@openrouter/ai-sdk-provider` (OpenRouter-compatible)
- **Auth**: Uses `apiKey: "anonymous"` for free models
- **Models API**: `https://api.kilo.ai/api/openrouter/models`

### Free Model Detection

Models are free when:
```typescript
parseFloat(pricing.prompt) === 0 && parseFloat(pricing.completion) === 0
```

### Model ID Format

Model IDs are used directly without prefix:
- `z-ai/glm-5:free`
- `deepseek/deepseek-r1-0528:free`
- `meta-llama/llama-3.3-70b-instruct:free`

## TypeScript Style

- Use strict typing; avoid `any`
- Use ESM `import` syntax
- Use `node:` specifier for Node built-ins
- Include `.js` extension for local imports

## Release Process

This project uses GitHub Actions for automated releases.

1. **Make changes with conventional commits:**
   ```bash
   git commit -m "feat: add new feature"  # Minor version
   git commit -m "fix: resolve bug"       # Patch version
   ```

2. **Push to main:**
   ```bash
   git push origin main
   ```

3. **Release Please creates PR:**
   - Review and merge the release PR
   - This triggers the publish workflow

4. **Verify:**
   - Check GitHub Actions
   - Verify npm: `npm view openkilo version`

## Configuration Files

| File | Purpose |
|------|---------|
| `~/.config/opencode/opencode.json` | OpenCode config (add `"openkilo"` to plugin array) |
| `~/.config/openkilo/auth.json` | Stored credentials |
| `~/.cache/openkilo/gateway-models.json` | Model registry cache |

## Testing Locally

```bash
# Build
bun run build

# Test in OpenCode
# Add path to opencode.json:
# "/Users/angansamadder/Code/opencode-plugins/openkilo"
```

## Notes for Agents

- Follow existing patterns and file naming
- Preserve ESM import style with `.js` extensions
- Update README.md when user-facing behavior changes
- Free models work without any API key (uses anonymous auth)

---
> Source: [AnganSamadder/openkilo](https://github.com/AnganSamadder/openkilo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
