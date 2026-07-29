## antigravity-ide-mobile

> **Generated:** 2026-03-10

# CLIENT (FRONTEND) KNOWLEDGE BASE

**Generated:** 2026-03-10
**Commit:** 9a1efd5
**Branch:** main

## OVERVIEW
Preact Vite frontend replacing the legacy vanilla JS public folder to handle REST and WebSocket communication.

## STRUCTURE
```
client/
├── src/components/ # UI components (TSX)
├── src/pages/      # Top level route components
├── src/hooks/      # Custom React hooks (API, WebSockets)
├── src/context/    # Global state context providers
├── src/chat/       # Chat specific components and logic
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Main App & Routing | `client/src/app.tsx` | Custom hash routing setup (`PanelRouter`) |
| Global State | `client/src/context/AppContext.tsx` | Global state definitions and the `useApp` hook |
| Network Requests | `client/src/hooks/useApi.ts` | Contains the `authFetch` wrapper |
| UI Updates | `client/src/chat/chatHandlers.ts` | Targeted UI updates using `morphdom` |

## CONVENTIONS
- **Framework**: We use Preact with Vite. Keep components lightweight and prefer TSX.
- **Routing**: We use custom hash routing instead of a heavy router library.
- **State**: Rely on `AppContext` for global state management. We bridge events using `useRef` where needed for performance.
- **UI Updates**: `morphdom` handles targeted UI updates without full DOM replacements.
- **Origin**: Code is ported from vanilla JS to Preact and TSX.

## ANTI-PATTERNS
- **Routing**: Don't use `react-router-dom` or other routing libraries. Stick to our custom hash router.
- **API**: Never use the native `fetch` API directly. Always import and use `authFetch`.
- **State**: Avoid heavy state management libraries like Redux. Rely on `AppContext` and local component state.
- **Updates**: Avoid full DOM replacements for dynamic lists. Use `morphdom` instead.

---
> Source: [mrkungfudn/antigravity-ide-mobile](https://github.com/mrkungfudn/antigravity-ide-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
