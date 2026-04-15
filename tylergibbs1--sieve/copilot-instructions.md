## sieve

> The browser for AI agents. Virtual mode for speed, real browser mode (Chrome/Lightpanda via CDP) when you need it.

# sieve

The browser for AI agents. Virtual mode for speed, real browser mode (Chrome/Lightpanda via CDP) when you need it.

## Quick reference

- **Runtime**: Bun
- **Language**: TypeScript (strict mode)
- **Tests**: `bun test`
- **Typecheck**: `bun run typecheck`
- **Benchmarks**: `bun benchmarks/core.ts`

## Architecture

```
src/
├── dom/          # Virtual DOM: nodes, parser (htmlparser2), serializer, HTMLRewriter preprocessing
├── css/          # CSS selector matching, computed styles (visibility/display)
├── a11y/         # Accessibility tree builder, LLM-optimized serializer, structured extraction
├── forms/        # Form state machine, validation, serialization
├── actions/      # Click, type, select simulation
├── cdp/          # Real browser via Chrome DevTools Protocol
│   ├── browser.ts    # CdpBrowser — launch/connect Chrome or Lightpanda
│   ├── page.ts       # CdpPage — full CDP page (screenshot, PDF, keyboard, HAR, etc.)
│   ├── session.ts    # WebSocket CDP client
│   ├── chrome.ts     # Chrome process launcher
│   ├── lightpanda.ts # Lightpanda process launcher
│   ├── tree.ts       # Chrome AX tree → sieve A11yNode conversion
│   └── protocol.ts   # CDP message types
├── navigation/   # URL routing, cookie jar, storage
├── snapshot/     # State capture, diff, restore, Bun.hash change detection
├── network/      # Pluggable fetchers: live HTTP, mock, disk replay (Bun.file/Bun.write)
├── persistence/  # SQLite persistence (bun:sqlite) for cookies, storage, snapshots
├── page.ts       # SievePage — virtual page abstraction
├── browser.ts    # SieveBrowser — virtual multi-page manager
└── index.ts      # Public API surface
```

## Bun-specific features used

- **bun:sqlite**: Persistent cookie jar, localStorage/sessionStorage, snapshot storage (WAL mode, in-memory or file-backed)
- **Bun.hash**: Wyhash for fast snapshot change detection, CRC32 for content-addressable IDs, SHA-256 for digests
- **HTMLRewriter**: Native streaming HTML preprocessing — strips scripts/styles/SVGs for agents, sanitizes XSS, extracts metadata
- **Bun.file / Bun.write**: Disk-backed replay fetcher records HTTP responses to filesystem
- **Bun.Glob**: Scanning replay recording directories
- **Snapshot testing**: `toMatchSnapshot()` for a11y tree regression tests

## Key design decisions

- Two modes: virtual (SieveBrowser/SievePage) for speed, CDP (CdpBrowser/CdpPage) for full browser
- Both modes share the same accessibility tree format (A11yNode), @ref addressing, and structured extraction
- Virtual pages are data structures (serializable TypeScript objects), not browser processes
- Accessibility tree is the primary agent interface
- Form state is tracked in WeakMaps separate from DOM attributes (value attr = default, WeakMap = current)
- CSS engine is minimal: visibility/display only, no layout
- htmlparser2 handles tokenization; our DOM is a thin layer on top
- CDP mode uses direct WebSocket to Chrome/Lightpanda — no Puppeteer/Playwright dependency
- All virtual interactions are synchronous except network operations and HTMLRewriter preprocessing
- Two parse paths: `parseHTML()` (sync, no preprocessing) and `parseHTMLAsync()` (with HTMLRewriter)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tylergibbs1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tylergibbs1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
