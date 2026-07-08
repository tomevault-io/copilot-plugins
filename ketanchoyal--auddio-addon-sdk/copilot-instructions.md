## auddio-addon-sdk

> **Generated:** 2026-02-16

# AUDDIO ADDON SDK (TypeScript) - PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-16
**Repo:** Separate GitHub repo
**Branch:** main

## OVERVIEW

TypeScript SDK for building Audiio addons. Provides types, interfaces, and utilities for creating addon servers that follow the Stremio addon protocol.

## TECH STACK

| Layer | Technology | Notes |
|-------|------------|-------|
| Language | TypeScript | Type definitions |
| Runtime | Node.js/Bun | Universal |
| Protocol | Stremio Addon | Types and interfaces |

## STRUCTURE

```
packages/auddio-addon-sdk/
├── src/
│   ├── types/          # TypeScript type definitions
│   ├── protocol/       # Stremio protocol types
│   ├── addon/          # Addon builder utilities
│   └── index.ts        # Main exports
├── package.json
└── tsconfig.json
```

## EXPORTS

- `AddonManifest` - Manifest type definition
- `AddonCatalog` - Catalog types
- `AddonStream` - Stream types
- `createAddon()` - Addon factory function

## ⚠️ GIT RULES (CRITICAL)

**This repo is INDEPENDENT. It has its OWN `.git` directory.**

### Rules:
1. **NEVER remove the `.git` folder** - This destroys connection to GitHub
2. **NEVER commit to a different repo** - Only commit within this directory
3. **Before committing, verify you're in the right repo**:
   ```bash
   cd packages/auddio-addon-sdk && git status
   ```

### Related Repos (DO NOT MIX):
| Directory | GitHub Repo |
|-----------|-------------|
| `app/` | https://github.com/ketanchoyal/audiio-app |
| `addons/audiobook-scraper/` | Separate repo |
| `addons/audiobook-unified-addon/` | Separate repo |
| `addons/debrid-stream/` | Separate repo |
| `packages/auddio-addon-sdk/` | **THIS REPO** |
| `packages/audioio_addon_sdk_dart/` | Separate repo |

## NOTES

- Published to npm as `@audiio/addon-sdk`
- Used by all addon packages
- Keep in sync with Dart SDK (audioio_addon_sdk_dart)

---
> Source: [ketanchoyal/auddio-addon-sdk](https://github.com/ketanchoyal/auddio-addon-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
