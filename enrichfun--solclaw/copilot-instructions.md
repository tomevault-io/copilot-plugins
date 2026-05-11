## solclaw

> This file is for **humans** and **AI coding agents** (Cursor, Copilot, Claude Code, etc.) working on Unknown Name Trader.

# AGENTS.md — Contributor & IDE-agent guide

This file is for **humans** and **AI coding agents** (Cursor, Copilot, Claude Code, etc.) working on Unknown Name Trader.

---

## IDE agent access

| Context | File access |
|---------|------------|
| **Cursor / VS Code agent attached to this folder** | Full — can search, read, edit files, run shell commands |
| **A chat tab not connected to the repo** | None — only sees what you paste. Point it at this file or README |
| **In-app Algo assistant** | `projectKnowledge` + **Live app state** at send (`buildChatContext.ts`); **Apply** commits proposed files to the user’s GitHub repo (`parseChatEdits.ts`, `ChatPanel.tsx`) |

---

## Quick commands

```bash
npm install
npm run dev        # runs workspace bundle first — Code sidebar loads project files without GitHub
npm run build      # → dist/ (bundle runs automatically)
npm run preview    # serve dist/ locally
npm run bundle-workspace   # optional — regenerate public/bundled-workspace/ only
```

---

## Architecture map

| Area | Primary paths | Notes |
|------|--------------|-------|
| App shell | `src/App.tsx`, `src/components/AppShell.tsx`, `src/components/AppTopChrome.tsx` | Top chrome + horizontal nav + resizable panels; `App.tsx` routes `/app` workspace, `/changelog` release notes (see `siteUrls.ts`), marketing landing otherwise |
| Top chrome | `src/components/AppTopChrome.tsx`, `src/hooks/useSetupProgress.ts` | Horizontal text nav + setup chip — add nav tabs when adding sidebar modes |
| Global state | `src/context/AppContext.tsx` | Single React context, localStorage-backed |
| Sidebar switcher | `src/components/DashboardSidebar.tsx` | Routes `sidebarMode` → panel component |
| Chart + mint | `src/components/CaChartPanel.tsx` | `lightweight-charts`, paper scalper trigger |
| PumpPortal WS | `src/lib/pumpPortalRealtime.ts`, `src/lib/pumpPortalConfig.ts`, `src/lib/solanaWalletSecret.ts` | Shared WebSocket, API key + optional wallet secret (derive pubkey for SOL chip) |
| Order book UI | `src/components/PumpOrderBook.tsx` | Live buy/sell tape |
| Candles | `src/lib/pumpCandles.ts` | REST candles + live MC merge; `/pump-api` on localhost (Vite proxy), direct URL on deployed sites |
| Paper scalper | `src/lib/scalperPaperEngine.ts`, `src/lib/scalperPaperConfig.ts` | Rules engine over live tape |
| Chat UI | `src/components/ChatPanel.tsx` | Tabs, markdown, SSE; live context + diff/Apply + deploy hooks |
| Chat context | `src/lib/buildChatContext.ts`, `src/lib/parseChatEdits.ts` | Live-state markdown; parses path-tagged fenced blocks, followups, algos |
| Chat streaming | `src/lib/streamChat.ts`, `src/lib/streamAnthropic.ts` | |
| Composer prompt | `src/lib/composerSystemPrompt.ts`, `src/lib/projectKnowledge.ts` | Docs + **browser session (GitHub)** note; `buildLiveContext` + custom instructions in `ChatPanel`; keep `projectKnowledge` accurate when paths change |
| Setup / LLM | `src/components/SetupPanel.tsx`, `src/lib/llmBackends.ts`, `src/lib/githubUpstreamDefaults.ts` | PumpPortal API key + trading-wallet secret (local); LLM; backup/import where applicable; GitHub PAT + **Fork & connect** helpers |
| Dev proxy | `vite.config.ts` (`LLM_PROXY_TARGETS`, `createLlmProxyMiddleware`) | Routes `/__proxy/llm/*` on localhost |
| Proxy helper | `src/lib/llmDevProxy.ts` | `resolveLlmApiUrl()` |
| GitHub workspace | `src/components/WorkspacePanel.tsx`, `src/lib/githubApi.ts` | **Code sidebar:** Monaco + tree; file ops via GitHub REST |
| Bundled IDE snapshot | `scripts/bundle-workspace.mjs` → `public/bundled-workspace/` (gitignored), `src/lib/bundledWorkspace.ts` | Default Code sidebar: no PAT; overlay in localStorage |
| Types | `src/types.ts` | Shared TS types |
| Styles / theme | `src/index.css` | CSS custom properties, utility classes |

---

## Key conventions

- **State**: everything global lives in `AppContext`. Add a `useState` + `useCallback`, persist with `localStorage` if needed, wire it into `useMemo` value + deps array.
- **Sidebar sections**: to add a new section — add a string to `SidebarMode` and `ActivitySection` in `AppContext`, add a nav tab in `AppTopChrome.tsx`, add a branch in `DashboardSidebar`, create the panel component.
- **API calls**: use `resolveLlmApiUrl(absoluteUrl)` so calls go through the localhost proxy on dev and direct on deployed URLs.
- **Imports**: use `@/` alias (maps to `src/`).
- **Styles**: Tailwind v4 utility classes + CSS custom properties from `index.css`. No inline `style` unless dynamic values.
- **Comments**: explain *why*, not *what*. No "// increment counter" style comments.

---

## Bundled assistant knowledge

`src/lib/projectKnowledge.ts` is injected into every in-app chat session. **Keep it accurate** when you:

- Move or rename files
- Add new sidebar sections or major features
- Change the Setup flow

The in-app assistant cannot read the filesystem — `projectKnowledge.ts` is its only source of truth.

---

## Secrets & safety

- **Never commit API keys.** PumpPortal, LLM, and GitHub PAT live in **browser `localStorage`** only.
- **`.env.local`** is in `.gitignore` — safe to use locally; never commit it.
- **PumpPortal trading-wallet private key** in Setup is stored in **browser `localStorage` only** (same risk model as API keys). Use a dedicated signer in production; Lightning trades still follow PumpPortal’s wallet tied to your API key unless you add local signing.

---

## PR checklist

- [ ] `npm run build` passes with zero errors
- [ ] If files moved/renamed → `projectKnowledge.ts` updated
- [ ] If architecture changed → this file (`AGENTS.md`) updated
- [ ] No API keys, emails, or personal tokens in any file

---
> Source: [Enrichfun/solclaw](https://github.com/Enrichfun/solclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
