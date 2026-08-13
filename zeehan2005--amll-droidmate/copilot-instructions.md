## amll-droidmate

> **本项目的 CLAUDE.md 已较大，优先复用当前上下文中已有的信息，而非重读文件。** 遵循以下原则：

# CLAUDE.md


## Token 节省策略

**本项目的 CLAUDE.md 已较大，优先复用当前上下文中已有的信息，而非重读文件。** 遵循以下原则：

- **简单操作说步骤** — 如果某个操作逻辑简单但需要大量 token（如批量搜索/读取多个小文件），直接描述步骤让用户自行处理，不要实际执行。
- **不 Read 刚写过的文件** — Edit/Write 失败会报错，成功即确认，无需再 Read 验证。
- **精准搜索** — 优先用 `Grep` 定位内容，而非 `Glob` + 逐个 `Read`。有明确目标时先试精确路径或关键词。
- **不读大全文** — 用 `limit`/`offset` 分段读取，或先用 `Grep` 定位行号再局部读取。
- **不重复已知** — 已从上下文或工具结果获得的信息不再二次确认。
- **不自动编译** — 不得运行任何编译命令，除非用户明确要求。

## Build & Run

**不要编译。** 用户自行编译，只需说明编译命令即可。只有用户明确要求时才运行编译。

## 编译缓存禁令

**禁止将任何问题归因于编译缓存。** 无论问题表现为何种形式，编译都是自动增量更新的，大量实例已充分证明缓存从未是问题的根源。任何以此为由的分析或建议都是错误的，应当立即排除。


## Architecture

This is **AMLL DroidMate** — an Android lyrics display app that shows Apple Music-style animated lyrics overlaid on any music source. It detects currently playing music via Android's `MediaSession` / notification listener, fetches lyrics from multiple online sources, and renders them with fluid animations in a WebView using the [`@applemusic-like-lyrics/core`](https://github.com/amll-dev/applemusic-like-lyrics) library.

### Two namespaces

| Namespace | Role |
|---|---|
| `io.github.zeehan2005.scoremuse` | ScoreMuse base: media detection, lyrics fetching/parsing, settings, theme, services, UI |
| `dev.amll.droidmate` | AMLL-specific: WebView lyrics view (`AMLLLyricsView`), TTML converter, WASM parser, AMLL settings |

### Key data flow

```
Music app playing → MediaInfoService (MediaSession callback) → MainViewModel
  → LyricsRepository.fetchLyricsAuto() (parallel search across QQ/Netease/Kugou/AMLL-DB)
  → UnifiedLyricsParser → TTMLConverter → UnifiedLyrics (cached via LyricsCacheRepository)
  → MainScreen → AMLLLyricsView (Compose WebView wrapper)
  → evaluateJavascript("window.updateLyrics(...)") → @applemusic-like-lyrics/core renders
```

### Core components

- **`MainActivity`** — Entry point. Initializes Timber logging, sets up edge-to-edge display, observes dynamic theme from album art, hosts `MainScreen`.
- **`MainViewModel`** — Central state manager. Holds `nowPlayingMusic`, `lyrics`, `songStructures`, `isLoading` as `StateFlow`s. Coordinates media listening, lyrics fetching, caching, and notification display.
- **`AMLLLyricsView`** — Composable wrapper around a native `WebView`. Injects lyrics JSON, playback time, album art, and motion configs via `evaluateJavascript`. Uses `WebViewAssetLoader` for secure local asset loading. Throttles time updates to 100ms intervals.
- **`LyricsRepository`** — Multi-source lyrics search/fetch. Parallel search across QQ Music (QRC/LRC), Netease (YRC/LRC), Kugou (KRC), and AMLL TTML DB. Includes sophisticated title/artist matching with Levenshtein distance, accent normalization, and version-keyword detection. All results are normalized to TTML via `TTMLConverter`.
- **`UnifiedLyricsParser`** — Format-detecting parser that dispatches to format-specific parsers (LRC, Enhanced LRC, QRC, KRC, YRC, TTML). All formats produce `UnifiedLyrics` (common data model) with `LyricLine`s containing text, translation, transliteration, per-word timing, and structural markers (isBG, isDuet, agent).
- **`ServiceLocator`** — Manual DI container providing singleton `HttpClient`, `LyricsRepository`, `LyricsCacheRepository`, and `WasmLyricParser`.
- **`DynamicThemeManager`** — Global observable `ColorScheme` extracted from album art by `AlbumColorExtractor` using AndroidX Palette.
- **`MediaInfoService`** — Polls `MediaController` for current playback info (title, artist, album art URI, position, play state).
- **`MediaListenerService`** — System `NotificationListenerService` for detecting which music app is playing.
- **`LyricNotificationManager`** — Persistent notification showing current lyric line, with optional lock screen display.
- **`LyricsCacheRepository`** — Local cache using SharedPreferences-backed key-value storage (song title + artist → TTML XML).

### Frontend (`frontend/`)

- **`src/main.tsx`** — Entry point. Initializes `DomLyricPlayer` from `@applemusic-like-lyrics/core`. Exposes `window.updateLyrics`, `window.updateTime`, `window.updateAlbumArt`, `window.setPaused`, `window.configureLyricMotion`, etc. for the Android bridge. Runs a `requestAnimationFrame` tick loop.
- **Vite config** — Library mode build (`amll.bundle.js`), CSS inlined into JS bundle, WASM + top-level-await plugins. Built output goes to `dist/` then Gradle copies to `app/src/main/assets/amll/`.
- **❗ `app/src/main/assets/amll/amll.bundle.js` 是构建产物，不要直接修改它。**
- **前端构建流程：** `cd frontend && npm run build` → `vite build` 生成 `dist/amll.bundle.js` → `postbuild` 自动运行 `scripts/patch-bundle.cjs` 修补产物。`build:android` 等效。Gradle 会从 `dist/` 复制到 assets。要测试修改：改 `main.tsx` → `npm run build` → 重新打包 APK。

### Data model

`UnifiedLyrics` is the universal lyrics representation, containing `LyricsMetadata` (title, artist, album, language, duration, source, `SongStructure` list) and `List<LyricLine>`. Each `LyricLine` has start/end time, text, optional translation/transliteration, per-word timing (`List<LyricWord>`), and flags (`isBG`, `isDuet`, `agent`). `SongStructure` marks sections (verse, chorus, bridge, intro, outro, etc.) identified from TTML metadata or inferred from lyric patterns.

## Logging conventions

This project uses **Timber** with strict conventions defined in `LOGGING_GUIDELINES.md`.

---
> Source: [Zeehan2005/AMLL-DroidMate](https://github.com/Zeehan2005/AMLL-DroidMate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
