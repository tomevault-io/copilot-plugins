## youtube-captions

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build (outputs to dist/cjs, dist/esm, dist/types.d.ts)
npm run build

# Publish — ALWAYS to both registries, never one (see Dual publishing below)
npm run publish:all

# Tests (Node.js built-in test runner)
npm test                # all tests
npm run test:unit       # pure-function tests (no network)
npm run test:integration  # live InnerTube calls
npm run test:channel    # live Data API calls (needs YOUTUBE_API_KEY)
```

## Dual publishing

This package is published to BOTH registries under the same name and version:

- **npmjs** (public) — what external users install; no auth needed.
- **GitHub Packages** (via the `@neurelectra:registry` mapping in internal `.npmrc` files) — what our own projects and Docker builds install.

Every release must go to both (`npm run publish:all`). Publishing to only one causes either
internal 404s on version bump (npmjs-only) or a stale public copy (GitHub-only).

## Architecture

This is a Node.js library (`@neurelectra/youtube-captions`) bundled via esbuild into dual CJS/ESM outputs.

### Source modules (`src/`)

| File | Responsibility |
|---|---|
| [src/index.js](src/index.js) | Public API re-exports only |
| [src/extractors.js](src/extractors.js) | Pure URL parsing: `extractVideoId`, `extractChannelIdentifier` |
| [src/metadata.js](src/metadata.js) | `getVideoMetadata` via YouTube oEmbed (no API key) |
| [src/transcript.js](src/transcript.js) | `getVideoTranscript`, `listCaptionTracks` via InnerTube API (no API key) |
| [src/content.js](src/content.js) | `getVideoContentDetails` via YouTube Data API v3 (requires `YOUTUBE_API_KEY`); supports batch IDs |
| [src/channel.js](src/channel.js) | `getChannelVideos` via YouTube Data API v3 (requires `YOUTUBE_API_KEY`) |
| [src/search.js](src/search.js) | `searchVideos` via YouTube Data API v3 (requires `YOUTUBE_API_KEY`); pagination + content enrichment |
| [src/types.d.ts](src/types.d.ts) | TypeScript declarations for all public exports |

### Key design decisions

**InnerTube client fallback chain** (`src/transcript.js`): Transcript fetching tries three InnerTube client configs in order — `IOS → ANDROID → WEB`. iOS and Android are preferred because they do not require the `poToken` attestation that the WEB client now demands.

**Caption track selection priority** (`getVideoTranscript`): manual preferred-lang → ASR preferred-lang → manual English → ASR English → first available.

**Caption format handling** (`fetchCaptionXml`): Forces `fmt=json3` on the timedtext URL. Parses `events[].segs[].utf8` from JSON3; falls back to XML `<text>` regex parsing if the response is a string.

**Dual CJS/ESM build** (`build.mjs`): esbuild bundles `src/index.js` into both formats. `axios` is marked external (not bundled). A `dist/cjs/package.json` with `{"type":"commonjs"}` is injected so Node.js treats the CJS output correctly.

### Logger convention

All async functions accept an optional `logger` option with Pino-style signature: `(level: 'debug'|'info'|'warn'|'error', context: object, msg: string) => void`. The library never logs when no logger is provided.

---
> Source: [neurelectra-oss/youtube-captions](https://github.com/neurelectra-oss/youtube-captions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
