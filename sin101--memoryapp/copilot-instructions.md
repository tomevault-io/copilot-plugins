## memoryapp

> 1. Run `npm test` from repo root after any backend change

# AGENTS.md — MemoryAPP Coding Rules

## After Every Implementation Session

1. Run `npm test` from repo root after any backend change
2. Run `npm run lint --prefix frontend` after any frontend change
3. Run `npm run build` to verify TypeScript compiles cleanly
4. **Update CLAUDE.md** — if new endpoints, components, patterns, or features were added/changed, update the relevant section
5. **Update memory files** — update `memory/features_implemented.md` (move items from Planned → Implemented), add new entries to `memory/feedback_patterns.md` if new constraints were discovered

## Code Style

- 2-space indentation in all files (TypeScript, JSX, JSON, CSS)
- TypeScript strict mode — no `any` without a comment explaining why
- Zod validation on every API request body in `server.ts`
- `useCallback` / `useMemo` required for all handlers and derived state in `App.jsx` and `CardGrid.jsx`

## Critical Constraints

### Pollinations.ai — BROWSER-SIDE ONLY
The Pollinations text and vision APIs (`text.pollinations.ai/openai`) MUST be called from the browser, not from the server. Server-side calls return 403 errors or a deprecation notice HTML page. Always guard parsed responses:
```js
if (content.includes('IMPORTANT NOTICE') || content.includes('deprecated')) return null;
```

### Tags — Store All, Display 10
Cards store up to 25 tags in `card.tags[]`. The UI displays only the top 10 as colored chips. Do NOT slice tags to 10 before storing — all 25 are needed for search, relation lines, deck creation, and suggestions.

### CardGrid — No react-window
CardGrid uses CSS grid (`auto-fill minmax(160px)`) instead of react-window. This is intentional: SVG relation lines require measuring actual DOM positions of card elements via `getBoundingClientRect`. Do not switch back to virtual lists.

### IndexedDB Encryption
The frontend encryption key (`encKey`) is a plain user-entered string. It lives in React state and `encKeyRef`. Never hash or transform it — it must be the same string on every load to decrypt stored cards.

### SSE
The server must send a heartbeat comment (`: heartbeat`) every 30s to keep SSE connections alive. Do not remove the `setInterval` in `server.ts` that sends heartbeats.

### Fuse.js Rebuild
The Fuse index in `App.jsx` must only rebuild when `cardSignature` changes, not on every card mutation. `cardSignature` is derived from `cards.map(c => id+title+desc+tags).join('|')`.

## File Responsibilities — Don't Mix Concerns

| Concern | Where it lives |
|---------|---------------|
| AI tag extraction (RAKE) | `contentAnalyzer.ts` |
| YouTube transcript | `contentExtractor.ts` + `/api/youtube-transcript` endpoint |
| URL metadata | `urlFetcher.ts` |
| Card/deck/link logic | `app.ts` |
| HTTP routing | `server.ts` |
| Browser AI calls (Pollinations) | `QuickAdd.jsx` |
| Offline storage | `App.jsx` (IndexedDB helpers at module level) |
| Tag display logic | `CardGrid.jsx` (top 10) and `CardDetailModal.jsx` (all) |

## Grep Tool Note

Use `path` parameter (not `file_path`) in the Grep tool:
```
Grep({ pattern: "...", path: "D:/git/MemoryAPP/src/server.ts", output_mode: "content" })
```

## Adding New API Endpoints

1. Add Zod schema for the request body
2. Add the route handler in `server.ts` (after similar endpoints, keep grouped by concern)
3. Add a cache if the operation is expensive and idempotent
4. Emit the appropriate SSE event if state changes
5. Update `CLAUDE.md` API table

## Adding New React Components

1. Create in `frontend/src/components/`
2. Use `memo()` if the component receives object/array props
3. All event handlers → `useCallback`; all derived values → `useMemo`
4. No class components; no Redux; no Context (App.jsx props only)
5. Update `CLAUDE.md` file map

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sin101) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
