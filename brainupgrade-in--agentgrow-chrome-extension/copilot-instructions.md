## agentgrow-chrome-extension

> **Version: v0.1.0** — submitted to Chrome Web Store for review.

# CLAUDE.md — AgentGrow Chrome Extension

**Version: v0.1.0** — submitted to Chrome Web Store for review.

Open-source AI browser assistant that **automates common browser tasks** — filling forms, writing email drafts, creating/editing/summarizing documents, navigating complex sites — to save users time. Connects to **any LLM provider** (OpenRouter, OpenAI, Anthropic, Groq, Ollama, or any OpenAI-compatible endpoint) via user-supplied API URL + key. No subscriptions. No vendor lock-in.

- **Extension homepage:** https://devops.gheware.com/agentgrow/
- **GitHub:** https://github.com/brainupgrade-in/agentgrow-chrome-extension

**Full design (architecture, data models, streaming, reliability, test suite):** `agentgrow.io-chrome-extension-design.md`  
**Reference studies:** `abacusai-chrome-ext.md`, `chrome-extensions-dev-best-practices.md`

---

## Project Goal

Build a Chrome extension (Manifest V3) that helps users automate repetitive browser tasks via AI — comparable to AbacusAI and Claude's browser extensions, but fully open source and LLM-provider-agnostic. Key differentiator: **live DOM read/write** on the adjacent tab — no screenshots, no snapshots.

### Core Use Cases
- **Form Automation** — auto-fill job applications, registrations, checkout forms, surveys
- **Email & Message Drafting** — compose emails in Gmail/Outlook, messages in Telegram/Slack
- **Document Tasks** — summarize articles, extract data (emails, links, tables, prices), create drafts
- **Site Navigation** — find information on complex sites, answer questions about page content
- **Data Extraction** — pull structured data from any page into usable formats

### Implemented Features (v0.1.0)
1. **Side panel chat with streaming** — persistent AI chat alongside any webpage; SSE streaming (OpenAI-compatible providers) + NDJSON streaming (Ollama) via named Port `llm-stream`
2. **Conversation persistence** — active conversation saved to `chrome.storage.local`, restored on panel reopen
3. **Smart auto-context** — auto-reads page content + text selection automatically; no manual toggles needed
4. **Multi-model dropdown** — switch models in 1 click; shows all preset models per provider + dynamically fetched models from any endpoint
5. **Chat UX** — New Chat button, copy message to clipboard, retry failed messages, timestamps on hover
6. **DOM write** — form fill (React/Angular/Vue compatible via native setter + `InputEvent` + full event suite), contenteditable support (Telegram, Slack, Gmail, Notion), text highlight, cursor insert
7. **DOM click & navigate** — click any button/link by CSS selector, scrollIntoView, React-compatible MouseEvent dispatch
8. **Smart selector resolution** — 5-level fallback: id → name → aria-label → placeholder → positional
9. **Action safety mode** — "Ask before acting" (default, shows approve/cancel UI) vs "Auto-act" with persistent risk warning
10. **Provider management** — add/edit/delete any OpenAI-compatible endpoint or Ollama; 7 built-in presets (OpenRouter, OpenAI, Anthropic, Groq, Google Gemini, Ollama, Custom); save confirmation toast; provider selection persisted to `chrome.storage.local`
11. **Ollama Cloud support** — api.ollama.com with API key; auto-shows key field for non-localhost Ollama URLs
12. **Private network support** — test connection fallback via tab execution for 192.168.*/10.*/172.16.* servers (bypasses service worker fetch restrictions)
13. **Friendly error messages** — 401 → "Invalid API key", 429 → "Rate limited", 404 → "Model not found", 5xx → "Server error"; retry button on all errors
14. **Error boundary** — wraps entire side panel; React crashes show recovery UI instead of blank screen
15. **Page reader** — extracts fillable fields + clickable elements with verified CSS selectors, contenteditable detection, UI noise filtering, line deduplication
16. **Dynamic model discovery** — "Fetch models from endpoint" button in provider form; fetches `/v1/models` (OpenAI-compatible) or `/api/tags` (Ollama); cached per-provider; chat header dropdown merges preset + fetched models; works with private network fallback
17. **In-page activity toast** — shadow-DOM toast at bottom of active tab showing "AgentGrow is working/filling/clicking/typing on this tab" with pulsing indicator + **Stop** button; appears during LLM streaming and DOM write operations; Stop aborts the stream and clears all toasts

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Build | Vite + `@crxjs/vite-plugin` |
| Language | TypeScript 5.x (strict) |
| UI | React 18 |
| State | Zustand (slice-per-domain) |
| Styling | Tailwind CSS + CSS variables |
| Markdown | react-markdown + remark-gfm + **rehype-sanitize** |
| Icons | Lucide React |
| Validation | **Zod** (all message payloads + provider URL validation) |
| Tests (unit) | Vitest + jsdom |
| Tests (e2e) | Playwright |
| API mocking | MSW (Mock Service Worker) |
| Crypto | Web Crypto API (built-in, no deps) |
| Package manager | pnpm |

---

## Chrome Extension Architecture (MV3)

```
Side Panel (React)  ←──sendMessage──→  Service Worker  ←─executeScript──→  Page DOM (reads)
Side Panel (React)  ←═══ Port ════════ Service Worker  ←──sendMessage──→  Content Script (writes)
                                             ↕
                                      User LLM endpoint
   Options.html                  (OpenRouter / Ollama / OpenAI / on-prem)
```

- `src/background/` — Service Worker: message router, LLM calls, **ProviderManager**, **KeyVault**, DOM relay, auth
- `src/sidepanel/` — React app: chat UI, settings, provider form, sign-in gate
  - `components/ErrorBoundary.tsx` — React error boundary wrapping entire side panel
- `src/content/` — Content script: **DOM write + activity toast** — form fill, click, highlight, insert, `__PING__` readiness check, shadow-DOM activity indicator with Stop button (reads done via `chrome.scripting.executeScript` inline from service worker)
- `src/options/` — Options page: privacy dashboard, audit log, about
- `src/core/` — Shared: provider interfaces, **dom types**, canonical types, utilities

**Key rules:**
- LLM API calls are made **only from the service worker** — never from content scripts
- API keys never reach content scripts, popup, or UI components
- One-shot operations use `sendMessage`; streaming uses a named `Port` (`llm-stream`)
- **All features are gated behind Google authentication** — unauthenticated users see only the sign-in screen
- **In-page activity toast** — shadow-DOM element injected by the content script; service worker drives it via `DOM_ACTIVITY_SHOW` / `DOM_ACTIVITY_HIDE` messages during LLM streaming and DOM write operations; Stop button sends `CHAT_STOP` to abort the stream

---

## LLM Provider System

Two adapters cover all supported providers — see design doc §5 for full implementation:

- `OpenAICompatibleProvider` — covers OpenRouter, OpenAI, Anthropic, Groq, Google Gemini, LM Studio, vLLM. SSE streaming with `AbortController` + connect/idle timeouts.
- `OllamaProvider` — local Ollama (NDJSON streaming, `/api/chat`, `/api/tags`)

All providers implement `ILLMProvider` (`src/core/providers/ILLMProvider.ts`): `complete()`, `stream(requestId, request)`, `listModels()`, `testConnection()`.

**Built-in presets (7):** OpenRouter · OpenAI · Anthropic · Groq · Google Gemini · Ollama (local) · Custom

Presets defined in `src/core/types/provider.ts` (`PROVIDER_PRESETS`). Each preset includes: base URL, curated model list, key placeholder, docs URL, and `requiresKey` flag. Managed via `ProviderManager.ts` (service worker) + `providerStore.ts` (Zustand, side panel).

**Dynamic model discovery:** The provider form has a "Fetch models from endpoint" button that sends `PROVIDER_LIST_MODELS` to the service worker. The SW fetches `/models` (OpenAI-compatible) or `/api/tags` (Ollama), parses the response, and caches model IDs per base URL in `chrome.storage.local` (key: `modelsCache:{baseUrl}`). The chat header dropdown merges preset models + cached fetched models. Works with private network fallback (same as `PROVIDER_TEST`).

---

## Source Layout

```
README.md         public-facing repo intro — install-unpacked instructions, security brief, issues link
dist/             built extension — load unpacked from here (ID: gdjoeliamfdblfefkcjcbipfcdoddebc)
store-listing.md  CWS listing copy + privacy disclosures
store-assets/     CWS graphics: icon-128, screenshots (5), promo tiles, marquee
agentgrow-v0.1.0.zip   CWS submission package (also attached to GitHub Release v0.1.0)
SHA256SUMS.txt    SHA-256 hash of the submission zip (also attached to GitHub Release v0.1.0)
app/              full extension source (Vite + crxjs, run all commands from here)
  scripts/
    zip.mjs           CWS zip builder (pnpm build:zip)
  src/
    background/
      index.ts          service worker — message router, LLM streaming (Port), DOM reads
                        (readPageDirect, readSelectionDirect), DOM clicks (clickDirect),
                        DOM fills (fillFormDirect), private network fallback, auth gate
      AuthService.ts    Google OAuth2 via chrome.identity
      ProviderManager.ts provider CRUD, storage, default management
      KeyVault.ts       AES-GCM-256 encrypted API key storage
    sidepanel/
      App.tsx           screen router (chat / settings / provider-add / provider-edit)
      components/
        ErrorBoundary.tsx React error boundary wrapping entire side panel
      views/
        ChatView.tsx        chat shell + smart auto-context (page/selection)
        SignInView.tsx       Google sign-in gate
        SettingsView.tsx     provider list + about links
        ProviderFormView.tsx add/edit provider with 7 presets
      hooks/
        useAuth.ts          auth state + session storage listener
        usePageContext.ts    DOM read/write hook (page context, selection, fill, highlight)
      store/
        providerStore.ts    Zustand store for provider list + active selection
      utils/
        messaging.ts        typed sendMessage() helper
    content/
      index.ts          DOM WRITE + activity toast — fill forms, click elements, highlight,
                        insert text, shadow-DOM "controlling this tab" indicator with Stop button
                        + __PING__ handler for content script readiness check
    options/
      main.tsx          settings page stub
    core/
      types/
        auth.ts         AuthSession, PublicUserInfo, AuthState
        messages.ts     MessageType enum, AgentGrowMessageSchema (Zod)
        provider.ts     ProviderConfig, PROVIDER_PRESETS (7 presets)
        dom.ts          StructuredPageContent, FormField, DomWriteResult, …
        conversation.ts Conversation, ConversationSummary
      utils/
        storage.ts      safeStorageSet (quota guard)
        url.ts          isAllowedProviderUrl, providerSecurityTier
  manifest.json   full MV3 manifest with oauth2 block
  package.json    pnpm deps
```

**Note:** No `popup/` directory — popup was removed; side panel is the primary UI.

**Run from `app/` directory for all commands below.**

---

## Common Commands

### Development

```bash
pnpm install                  # install dependencies
pnpm dev                      # Vite watch mode (outputs to dist/)
pnpm build                    # production build
pnpm build:zip                # create CWS-ready zip
pnpm audit                    # check for vulnerable dependencies (run before every release)
```

### Load Extension in Chrome

1. `pnpm build` (or `pnpm dev` for hot-reload via crxjs)
2. `chrome://extensions` → Developer Mode → Load Unpacked → select `dist/` (repo root)
3. Pin the extension to toolbar

### Testing

```bash
pnpm test                     # run all unit tests (Vitest)
pnpm test:watch               # watch mode
pnpm test:coverage            # with coverage report
pnpm test:e2e                 # Playwright e2e (requires pnpm build first)
pnpm test:e2e --headed        # with visible browser
pnpm typecheck                # tsc --noEmit
pnpm lint                     # ESLint
pnpm lint:fix                 # ESLint --fix
```

### Debugging

- **Side panel**: Right-click panel → Inspect
- **Service worker**: `chrome://extensions` → click "Service Worker" link
- **Content script**: DevTools on page → Sources → Content Scripts
- **Errors**: Red "Errors" button on `chrome://extensions`

---

Full annotated file tree: design doc §12.

---

## Manifest Permissions

```json
"permissions": ["storage", "tabs", "tabGroups", "sidePanel", "activeTab",
                "scripting", "alarms", "contextMenus", "notifications", "identity"],
"host_permissions": ["<all_urls>"],
"optional_host_permissions": ["<all_urls>"],
"oauth2": {
  "client_id": "1088798291609-e95e1put9gp75s25o8c25l65kr9defv8.apps.googleusercontent.com",
  "scopes": ["openid", "email", "profile"]
}
```

- `identity` permission enables `chrome.identity` API (Google OAuth2 flow)
- `oauth2` block ties the extension to a Google Cloud Console OAuth2 client
- **Extension ID: `gdjoeliamfdblfefkcjcbipfcdoddebc`** (loaded unpacked from `dist/`)
- `host_permissions` includes `<all_urls>` — needed for `chrome.scripting.executeScript` inline page reads on any tab

### Google Cloud Console Setup (completed — branding published April 2026)

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → Create project `agentgrow`
2. APIs & Services → OAuth consent screen → External → fill App name, support email, scopes: `openid email profile`
3. APIs & Services → Credentials → Create credentials → OAuth client ID → Application type: **Chrome App**
4. Application ID: `gdjoeliamfdblfefkcjcbipfcdoddebc`
5. Copy the `client_id` → paste into `manifest.json` `oauth2.client_id`
6. For production CWS publish: update the ID to the published extension's ID and recreate credentials

---

## Security Rules

Hard rules enforced in every PR. See design doc §9 for threat model and full CSP.

### A. Permissions (Least Privilege)
1. `host_permissions` is `["<all_urls>"]` — required for `chrome.scripting.executeScript` page reads on any tab. This was moved from `optional_host_permissions` because inline script injection for DOM extraction needs broad host access.
2. Use `activeTab` over host permissions wherever possible.
3. Do NOT add `externally_connectable` without a documented need.
4. `web_accessible_resources` stays `[]` unless restricted to specific trusted origins.
5. Do NOT register `runtime.onMessageExternal`.

### B. API Keys & Storage
6. API keys encrypted at rest (AES-GCM-256, `KeyVault.ts`). Never plaintext in `chrome.storage`.
7. Keys never leave the service worker — no raw key values in content scripts, popup, or options.
8. No sensitive data in `chrome.storage.sync` — sync goes to Google servers.

### C. Network
9. HTTPS required for all remote provider URLs. `http://` allowed only for localhost or private IP ranges.
10. Re-validate URL at call time, not just at save time.

### D. Message Passing & Input Validation
11. Treat all content script messages as untrusted. Validate `sender.tab` before acting on privileged operations.
12. Validate all message payloads with Zod schemas before use.
13. Never send sensitive data back to content scripts.

### E. DOM & XSS Prevention
14. Never use `innerHTML`, `outerHTML`, or `document.write()` anywhere.
15. Never use `dangerouslySetInnerHTML`. Use `rehype-sanitize` in the react-markdown pipeline.
16. Wrap all untrusted page content with `=== PAGE CONTEXT ===` delimiters. Truncate to `MAX_CONTEXT_CHARS`.

### F. Content Script Safety
17. Content script handles **write operations + activity toast** (form fill, click, highlight, insert, in-page indicator). DOM reads are performed via `chrome.scripting.executeScript` inline functions from the service worker (`readPageDirect`, `readSelectionDirect`) — no content script involvement for reads.
18. Sanitize DOM text before sending to the service worker. Strip `<script>`, `<style>`, event attributes.

### L. Action Safety
40. **"Ask before acting" mode (default)** — all DOM write operations (fill, click, submit) show an approve/cancel UI in the chat before execution. User must explicitly approve each action.
41. **"Auto-act" mode** — executes DOM writes immediately. A persistent risk warning banner is shown whenever this mode is active. Preference stored in `chrome.storage.local`.
42. Mode toggle is accessible from the chat input area. The setting persists across sessions.

### K. DOM Write & Click Safety
34. Form fills target `<input>`, `<textarea>`, `<select>`, and `contenteditable` elements. Click targets include `<button>`, `<a>`, `<input type="submit">`, and any element with a click handler.
35. Validate CSS selectors with a try/catch before querying (`document.querySelector` throws on bad selectors).
36. Use the native value setter + dispatching `InputEvent` + full event suite (`focus`, `input`, `change`, `blur`) for React/Angular/Vue-compatible form fills — never `el.setAttribute('value', …)`.
37. `DOM_FILL_FORM`, `DOM_CLICK`, and other DOM write message types originate from the service worker (after auth gate) — never directly from the page.
38. Text highlighted via `<mark class="agentgrow-highlight">` must be clearable; never remove marks without `parent.normalize()` to avoid orphan text nodes.
39. `insertTextAtCursor` uses `document.execCommand('insertText')` only for contenteditable — deprecated but still the only safe cross-framework method; guard with `isContentEditable` check.

### M. In-Page Activity Toast
43. Activity toast uses a **shadow-DOM** element with a closed `ShadowRoot` — isolated from host page CSS and cannot be manipulated by page scripts.
44. Toast is controlled exclusively by `DOM_ACTIVITY_SHOW` / `DOM_ACTIVITY_HIDE` messages from the service worker — never by page JavaScript.
45. The Stop button sends `CHAT_STOP` via `chrome.runtime.sendMessage` — content script cannot directly access the port or abort controller.
46. `CHAT_STOP` from content scripts is treated as privileged (goes through the authenticated message router). It aborts all in-flight `AbortController` instances and broadcasts `DOM_ACTIVITY_HIDE` to all tabs.

### G. Content Security Policy
19. CSP: `"script-src 'self'; object-src 'none'; base-uri 'none';"` — no eval, no inline, no CDN, no base-tag injection.

### H. Dependency Security
20. Run `pnpm audit` in CI on every push. Block merges on high/critical vulnerabilities.
21. No CDN-loaded scripts. All dependencies bundled at build time.
22. Minimise dependencies — prefer built-in Web APIs.

### J. Authentication

28. **All features gated behind Google auth.** The service worker checks auth state before processing any message. Unauthenticated callers receive `{ success: false, error: 'UNAUTHENTICATED' }`.
29. **Auth token lives in `chrome.storage.session` only** — cleared when the browser closes. Never in `local` or `sync` storage.
30. **`chrome.identity.getAuthToken({ interactive: false })` on every service worker wake** — verify token is still valid before handling any request. If expired or absent, set auth state to signed-out.
31. **Token never leaves the service worker** — UI components receive only `{ email, name, picture, isAuthenticated: true }`. Raw Google OAuth token is never sent to the side panel, popup, or content scripts.
32. **Sign-out revokes the token** via `chrome.identity.removeCachedAuthToken` + `chrome.identity.revokeToken`. Clears all user-specific data from `chrome.storage.session`.
33. **Incognito mode**: `chrome.identity` does not work in incognito. Detect via `chrome.extension.inIncognitoContext` and show an explicit `"Sign-in not available in Incognito"` message.

### I. Chrome Web Store Compliance
23. Single purpose — no unrelated features.
24. Privacy policy at `https://devops.gheware.com/agentgrow/` before CWS submission.
25. Data handling disclosures must match the code exactly.
26. No telemetry — zero data to AgentGrow servers.
27. Enable 2FA on the CWS publisher account.

### Quick Security Checklist (pre-PR)

- [ ] No `innerHTML` / `dangerouslySetInnerHTML` / `document.write` introduced
- [ ] No new permissions added to `manifest.json` without justification
- [ ] No API keys passed to content scripts or stored in sync storage
- [ ] All new `onMessage` handlers validate sender and payload shape
- [ ] All provider URLs validated (HTTPS or localhost/private IP only)
- [ ] `web_accessible_resources` still `[]` or restricted to specific origins
- [ ] `pnpm audit` passes with no high/critical issues
- [ ] react-markdown uses `rehype-sanitize` for any HTML output
- [ ] No remote scripts / CDN URLs in code
- [ ] Auth check (`ensureAuthenticated()`) present in every new message handler
- [ ] Raw Google OAuth token never sent outside the service worker
- [ ] DOM write targets validated (only form elements, valid CSS selector, guarded with try/catch)
- [ ] New DOM_* message types added to `DOM_RELAY_TYPES` set in service worker
- [ ] Activity toast shows during new DOM operations (wrap with `setActivity()`)
- [ ] Activity toast cleaned up in `finally` blocks — no leaked "controlling" state

---

## Stability & Reliability Rules

The extension must degrade gracefully at every layer — not silently. See design doc §10 for full architecture, diagrams, and implementation patterns.

### Core Rules

| Area | Rule |
|------|------|
| Service Worker | `ensureInitialized()` at the top of every event handler — never assume in-memory state survived |
| Service Worker | No module-level mutable state. All state loaded from `chrome.storage` on wake |
| Streaming keepalive | Alarm (`stream-keepalive`, every 24s) active while any stream is in progress |
| Streaming transport | Use named `Port` (`llm-stream`) — not `sendMessage` — for all token streaming |
| Port disconnect | `useServiceWorkerPort` reconnects with linear back-off; marks in-flight messages as errored |
| Request cancellation | Every `fetch` has an `AbortController`. Cancel on: Stop button, port disconnect, `onSuspend` |
| Timeouts | 15s connect timeout + 30s idle-token timeout on every LLM fetch |
| Storage writes | All writes through `safeStorageSet` (quota check before write, rotate at 8 MB) |
| Schema changes | `runMigrations()` in `onInstalled` + every SW wake. All migrations idempotent |
| React crashes | `ErrorBoundary` wraps every view independently. Crashes logged locally — no telemetry |
| Loading states | Every async operation has a skeleton/spinner. No blank areas while waiting |
| Content script timeout | 5s timeout on page extraction. Non-fatal — chat continues without page context |
| Provider health | `HealthChecker` caches status (5 min TTL). Shown in `NetworkStatusBar` before first message |
| Token render | `useStreamBuffer` batches tokens at 50ms — no per-token React state updates |
| Message list | `react-window` virtualisation for 500+ messages. Auto-scroll only when user is at bottom |
| Offline detection | `useNetworkStatus` disables input + shows banner when offline. Localhost providers exempt |
| Context invalidation | `chrome.runtime.id` polled every 10s. Non-dismissable reload banner on invalidation |
| Activity toast cleanup | `setActivity(hide)` in `finally` blocks + port `onDisconnect`. `CHAT_STOP` broadcasts clear to all tabs |

### Stability Checklist (pre-PR)

- [ ] All service worker state reads from storage, not module-level variables
- [ ] Every LLM fetch has an `AbortController` and a connect timeout
- [ ] Port disconnect handler calls `markStreamingMessagesAsError`
- [ ] New storage writes go through `safeStorageSet`
- [ ] New async operations have loading states (skeleton or spinner)
- [ ] New React views are wrapped in `<ErrorBoundary>`
- [ ] Content script failures time out gracefully (never block chat)
- [ ] Token streaming goes through `useStreamBuffer` (no per-token state updates)
- [ ] Activity toast cleaned up on port disconnect, stream abort, and CHAT_STOP

---

## Design System

- **Theme**: dark, terminal-refined
- **Accent**: `#22d3a8` (emerald-teal)
- **Background**: `#0e0e11` base / `#16161d` surface
- **Fonts**: JetBrains Mono (mono/code) · DM Sans (UI) · Inter (body/chat)
- **Transitions**: 150ms max
- **Side panel width**: 400px (Chrome default)

Full tokens in `src/assets/styles/tokens.css`. Full UI design in design doc §7.

---

## Trust Features (summary)

First-class features — not afterthoughts. Full specification in design doc §6.

| ID | Feature | Where |
|----|---------|-------|
| T1 | Privacy Dashboard — shows exactly what is stored and where | Options → Privacy |
| T2 | Live Network Indicator — shows endpoint + status per request | Side panel status bar |
| T3 | Permission Explainer — each permission in plain English | Options → About Permissions |
| T4 | Open Source Verification — GitHub link + version + build hash | Side panel header, Options → About |
| T5 | Data Export & Deletion — one-click JSON export, full wipe | Options → Privacy |
| T6 | Audit Log — API call metadata only (no content), 500 entries | Options → Privacy → Network Log |
| T7 | Provider Security Badge — HTTPS/local/warning indicator | Provider list + side panel header |
| T8 | First-Run Transparency Screen → Google Sign-In — no dark patterns, no email capture | On install |
| T9 | Reproducible Build Badge — SHA-256 of zip published in CI | GitHub Releases |
| T10 | CWS Compliance Page — privacy policy, data disclosures | https://devops.gheware.com/agentgrow/ |

---

## Coding Conventions

| Kind | Convention |
|------|-----------|
| Class files | `PascalCase.ts` |
| Utility files | `camelCase.ts` |
| React components | `PascalCase.tsx` |
| Types / interfaces | PascalCase |
| Constants | `SCREAMING_SNAKE` |
| Message types | `SCREAMING_SNAKE` string enum |

- **Named exports everywhere** except React component files (`.tsx`)
- **Import order**: builtins → external packages → chrome → internal absolute → relative
- **Wrap `chrome.*` callbacks** in promisified helpers in `src/core/utils/chrome.ts`. No raw callbacks in business logic.
- **No default exports** except `.tsx` component files

---

## Development Phases

### Phase 1 — MVP (v0.1.0 — submitted to CWS for review)
- ✅ Google authentication (chrome.identity OAuth2)
- ✅ Provider management (7 presets: OpenRouter, OpenAI, Anthropic, Groq, Gemini, Ollama, Custom)
- ✅ Settings UI — inline in side panel (gear icon left of logo)
- ✅ Provider/model picker in chat header (OpenCode-familiar `provider › model`)
- ✅ Live DOM read — structured extraction (headings, fillable fields with selectors, clickable elements, code blocks, links, selection, contenteditable detection, UI noise filtering)
- ✅ Live DOM write — form fill (React/Angular/Vue-compatible), contenteditable (Telegram/Slack/Gmail/Notion), text highlight + clear, cursor insert
- ✅ DOM click & navigate — click buttons/links by CSS selector, scrollIntoView, React-compatible MouseEvent dispatch
- ✅ Smart selector resolution — 5-level fallback (id → name → aria-label → placeholder → positional)
- ✅ Action safety mode — "Ask before acting" (default) vs "Auto-act" with persistent risk warning
- ✅ Smart auto-context — auto-reads page + selection, no manual toggles
- ✅ LLM streaming chat — SSE (OpenAI-compatible) + NDJSON (Ollama) via named Port `llm-stream`
- ✅ Conversation persistence — active conversation saved to chrome.storage.local, restored on panel reopen
- ✅ Chat UX — New Chat, copy message, retry failed, timestamps on hover, friendly error messages (401/429/404/5xx) with retry button
- ✅ Multi-model dropdown — switch models in 1 click, shows all preset models per provider
- ✅ Error boundary wrapping entire side panel
- ✅ Save confirmation toast after provider test
- ✅ Provider selection persisted to chrome.storage.local
- ✅ Private network support — test connection fallback via tab execution for 192.168.*/10.*/172.16.* servers
- ✅ Ollama Cloud support — api.ollama.com with API key, auto-shows key field for non-localhost
- ✅ Page reader — fillable fields + clickable elements with verified CSS selectors, contenteditable detection, UI noise filtering, line deduplication
- ✅ CWS submission — v0.1.0 zip uploaded, store listing + screenshots + privacy disclosures prepared
- ✅ GitHub Release v0.1.0 — tag pushed, zip + SHA256SUMS.txt attached (https://github.com/brainupgrade-in/agentgrow-chrome-extension/releases/tag/v0.1.0)
- ✅ README.md — public-facing install-unpacked instructions, security brief, feedback/issues link
- ✅ Dynamic model discovery — "Fetch models from endpoint" for any OpenAI-compatible or Ollama provider; cached per-base-URL; merged into chat header dropdown
- ✅ In-page activity toast — shadow-DOM indicator on active tab during LLM streaming and DOM writes, with Stop button to abort the stream
- ⬜ Multi-tab group summary
- ⬜ Prompt templates
- ⬜ Full test suite + reliability e2e
- ⬜ CI/CD pipeline

### Phase 2 — Agentic
Context menu integration, AI form-filling via chat command, structured data extraction to clipboard/JSON, tab group research briefs.

### Phase 3 — Power Features
Multi-provider routing, prompt chaining, scheduled tasks, Firefox port, import/export, optional shared template library.

---

## Release Process

1. Bump version in `manifest.json` and `package.json` (must match)
2. Update `CHANGELOG.md`
3. `pnpm typecheck && pnpm lint && pnpm test:coverage && pnpm audit`
4. `pnpm build:zip` → verify SHA-256
5. Tag `v1.2.3` → CI publishes GitHub Release with zip + `SHA256SUMS.txt`
6. Upload zip to CWS Developer Dashboard
7. Update version badge on `devops.gheware.com/agentgrow/`

**Semver guidance:** PATCH = bug fixes; MINOR = new features; MAJOR = manifest permission changes (triggers CWS re-review) or breaking storage format changes.

---

## CWS Store Assets

All assets for Chrome Web Store submission are prepared and included in the repo:

| Asset | Location |
|-------|----------|
| Store listing copy + privacy disclosures | `store-listing.md` |
| Store icon (128x128) | `store-assets/store-icon-128.png` |
| Screenshots (5) | `store-assets/screenshot-{1..5}.png` |
| Promo tiles (440x280, 920x680, 1400x560) | `store-assets/promo-tile-*.png` |
| Marquee promo (1400x560) | `store-assets/marquee-promo-1400x560.png` |
| Submission zip | `agentgrow-v0.1.0.zip` |
| Zip hash | `SHA256SUMS.txt` |
| Zip build script | `app/scripts/zip.mjs` |

---

## Key References

- Chrome Extension MV3 docs: https://developer.chrome.com/docs/extensions
- Chrome Extension Security guide: https://developer.chrome.com/docs/extensions/mv3/security
- Chrome Web Store policies: https://developer.chrome.com/docs/webstore/program-policies
- OWASP Browser Extension Vulnerabilities: https://cheatsheetseries.owasp.org/cheatsheets/Browser_Extension_Vulnerabilities_Cheat_Sheet.html
- crxjs Vite plugin: https://crxjs.dev/vite-plugin
- Playwright extension testing: https://playwright.dev/docs/chrome-extensions
- OpenRouter API: https://openrouter.ai/docs
- Ollama API: https://github.com/ollama/ollama/blob/main/docs/api.md
- rehype-sanitize: https://github.com/rehypejs/rehype-sanitize
- Zod validation: https://zod.dev

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brainupgrade-in) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
