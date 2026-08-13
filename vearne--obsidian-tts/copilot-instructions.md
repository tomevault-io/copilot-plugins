## obsidian-tts

> npm install              # install dependencies

# AGENTS.md — obsidian-tts

## Build & Dev Commands

```bash
npm install              # install dependencies
npm run dev              # esbuild watch mode (outputs main.js, rebuilds on save)
npm run build            # tsc typecheck + production bundle (minified, no sourcemap)
```

- `npm run build` runs `tsc -noEmit -skipLibCheck` **before** bundling — type errors block the build.
- Output: `main.js` (gitignored), `styles.css`, `manifest.json` — these three files are the distributable plugin.
- esbuild config: `esbuild.config.mjs` — entry `src/main.ts`, platform `node`, format `cjs`, target `es2018`.

## Testing & Lint

- **No test framework** is configured. There are no test files, no jest/vitest config.
- ESLint devDeps are installed (`@typescript-eslint/*`) but there is no `.eslintrc` config file and no lint script.
- TypeScript strict mode is enabled (`strictNullChecks: true`, `noImplicitAny: true`).

## Architecture

### Plugin Wiring (src/main.ts)

`ObsidianTtsPlugin` extends Obsidian's `Plugin` class. `onload()` wires together:
- `TTSEngine` — orchestrates text → audio synthesis
- `PlaybackManager` — plays ArrayBuffer chunks via HTMLAudioElement
- `QueueManager` — FIFO queue of notes/text to read sequentially
- `FloatingPlayer` — draggable floating play/pause/stop widget
- `QueuePanel` — sidebar panel showing queued items
- Settings persistence via `this.loadData()` / `this.saveData()` (Obsidian's JSON storage)

### TTS Provider System (src/tts/)

Strategy pattern with `TTSProvider` interface:
- `provider.ts` — interface (`synthesize`, `getMaxChunkSize`, `listVoices?`)
- `registry.ts` — factory: maps `ProviderId` → provider instance
- `engine.ts` — coordinates: preprocess text → chunk → synthesize each chunk sequentially → return `ArrayBuffer[]`
- `chunker.ts` — splits text at sentence/paragraph boundaries, with special GBK-byte-based chunking for Baidu

**8 providers** in `src/tts/providers/`:
| File | ProviderId | API Key Required | Audio Format |
|------|-----------|-------------------|-------------|
| edge.ts | `edge` | No (WSS direct or proxy) | MP3 |
| openai.ts | `openai` | Yes | MP3 |
| openai.ts | `openai-compatible` | Optional | MP3 |
| azure.ts | `azure` | Yes | MP3 |
| google.ts | `google` | Yes | MP3 |
| elevenlabs.ts | `elevenlabs` | Yes | MP3 |
| zhipu.ts | `zhipu` | Yes | WAV/PCM |
| baidu.ts | `baidu` | Yes (apiKey+secretKey) | MP3 |

### Key Directories

```
src/
  main.ts              # Plugin class, commands, ribbon icon, file menu
  tts/
    engine.ts           # Synthesis orchestration (chunk → synthesize → ArrayBuffer[])
    provider.ts         # TTSProvider interface + utility functions
    registry.ts         # Provider factory (switch on ProviderId)
    chunker.ts          # Text splitting at sentence/paragraph boundaries
    providers/          # One file per TTS backend (edge, openai, azure, google, elevenlabs, zhipu, baidu)
  audio/
    playback-manager.ts # HTMLAudioElement playback, pause/resume, segment chaining
    queue-manager.ts    # FIFO queue for sequential note reading
  settings/
    types.ts            # ObsidianTtsSettings interface, DEFAULT_SETTINGS, voice lists, ProviderId type
    setting-tab.ts      # Obsidian PluginSettingTab UI
  ui/
    floating-player.ts  # Draggable floating player widget
    queue-panel.ts      # Queue display panel
  file/
    mp3-export.ts       # Export MP3 to vault, embed ![[link]] in note (desktop only)
  text/
    preprocessor.ts     # Markdown → plain text conversion, frontmatter/code/link stripping
  utils/
    http.ts             # Wrapper around Obsidian's requestUrl with logging
    logger.ts           # Debug logging with secret masking, controlled by settings
```

## Critical Build Quirks

### esbuild Aliases (esbuild.config.mjs)

The build aliases `edge-tts-universal` to its CJS dist file and `isomorphic-ws` to `ws`. This bypasses the package's `browser` field to force Node.js code paths (needed for WSS WebSocket headers). The `packages: "bundle"` setting forces `edge-tts-universal` into the bundle to avoid Obsidian runtime `require()` failures.

### External Modules

`obsidian`, `electron`, and CodeMirror packages are marked external — they come from the host app at runtime. Node builtins are also external via the `builtin-modules` package.

## Platform Awareness

- **Desktop vs Mobile**: Edge TTS direct WSS connection is **desktop-only** (`Platform.isMobile` check). Mobile falls back to proxy mode automatically.
- **MP3 export** is desktop-only (`Platform.isMobile` guard in setting-tab and commands).
- Baidu TTS has a special chunker that counts GBK bytes instead of characters.

## Provider Patterns

- **Edge**: Two modes — WSS direct (via `edge-tts-universal` Communicate class) and HTTP proxy (OpenAI-compatible `/audio/speech` endpoint). Auto mode tries direct first, falls back to proxy on failure.
- **Baidu**: Two-key auth (apiKey + secretKey), requires token refresh before each synthesis call.
- **Zhipu**: Returns WAV format by default (not MP3). The `getAudioFormat()` helper in mp3-export.ts handles the format distinction.

## Adding a New Provider

1. Create `src/tts/providers/newprovider.ts` implementing `TTSProvider` interface
2. Add provider ID to `ProviderId` union type in `src/settings/types.ts`
3. Add settings section to `ObsidianTtsSettings` interface + `DEFAULT_SETTINGS`
4. Add voice list constants if applicable
5. Add `PROVIDER_LABELS` entry
6. Register in `ProviderRegistry.getProvider()` switch statement
7. Add settings UI in `ObsidianTtsSettingTab`

## Conventions

- UI strings are in Chinese (zh-CN) — commands, notices, settings labels
- All provider HTTP calls go through `loggedRequest()` in `src/utils/http.ts` (wraps Obsidian's `requestUrl`)
- Debug logging is gated by `settings.enableDebugLog` (default: `true`)
- Secrets (API keys, tokens) are masked in logs via `maskSecret()`
- `formatError()` in logger.ts handles non-Error thrown values safely

---
> Source: [vearne/obsidian-tts](https://github.com/vearne/obsidian-tts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
