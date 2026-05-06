## architecture

> - `background/aid/api/` — one file per GraphQL query or mutation (never inline queries elsewhere)

# MCA Architecture

## Folder Conventions

- `background/aid/api/` — one file per GraphQL query or mutation (never inline queries elsewhere)
- `background/openrouter/api/` — one file per OpenRouter API call
- `background/protocol.js` — single source of truth for all message types between popup, content, and background
- `background/types.js` — all JSDoc typedefs live here, import via `@typedef {import('./types.js').TypeName}`
- `background/storage.js` — all chrome.storage access goes through these helpers
- `background/constants.js` — shared constants (API URLs, defaults). Background modules only — content scripts inline their own

## Service Worker Rules

- **Never store persistent state in module-scope variables.** Service workers go dormant after ~30s. Use `chrome.storage` via `storage.js`
- **Register all listeners synchronously** at the top level of `background/index.js` (Chrome requirement)
- **Message routing** lives in `background/index.js` as a switch statement. Import protocol constants, not raw strings
- **Handler implementations** live in `background/handlers.js` — one exported function per message type

## Content Script Rules

- Content scripts **cannot use ES module imports**. Inline any needed constants
- Reference `background/protocol.js` in comments when using message type strings
- Token extraction uses `window.postMessage` between `inject.js` (page context) and `index.js` (content script context)

## Error Handling

- Async operations return `{ ok: boolean, value?, error? }` (the `Result` type from `types.js`)
- GraphQL errors are caught in `gql.js` and thrown as standard Errors
- Never let errors silently disappear — log with `[MCA]` prefix

## Agent Workspace

- `dev/agents/` contains markdown files for agents to track work across sessions
- **Read before starting**: `session-notes.md` (what was done last), `decisions.md` (why things are the way they are)
- **Update when done**: `session-notes.md` with what you did and what's next
- **Record findings**: `investigations.md`, `api-testing.md`, `api-traffic.md`, `token-notes.md`
- **Track issues**: `bugs.md`
- **Create new files** for any topic that doesn't fit existing files, and update `README.md`
- **Undocumented work is lost work** — the next agent starts from zero without notes

## Popup Module Pattern

- `popup.html` loads `popup.js` as `type="module"` — all popup files use ES module imports
- `popup/helpers.js` — shared utilities (`$`, `send`, `setDot`, `filterBranches`), `ctx` (transient state), agent a11y helpers
- `popup/render.js` — pure rendering functions (`makeField`, `renderPlotFields`, `buildCardItem`, `updateCardCount`, `setupCardsToggle`)
- `popup/events.js` — optimistic UI event handlers (`mca:branch-added`, `mca:branch-deleted`, `mca:branch-renamed`, `mca:navigate`)
- `popup/clone.js` — clone panel logic (self-contained, registers own event listeners at module load)
- `popup/delete.js` — delete confirmation dialog + API call, dispatches `mca:branch-deleted` event
- `popup/tree.js` — tree view navigation: lazy-loading hierarchy, expand/collapse, direct navigation via `mca:navigate`
- `popup/popup.js` — entry point: init, navigation, branch list, delegation, broadcasts
- **Cross-module state**: `ctx` from `helpers.js` holds `{ scenario, branches, state }` — all popup modules read/write it
- **Cross-module signals**: Use `CustomEvent("mca:...")` to avoid circular imports. clone.js → `mca:branch-added`, delete.js → `mca:branch-deleted`, tree.js → `mca:navigate`, events.js listens and coordinates
- **Test harness access**: popup.js exposes `ctx`, `openClonePanel`, `closeClonePanel`, `rebuildBranchList`, `deleteBranch` on `window`
- **New features** (settings, generation, diff) each get one new popup module file — never bolt onto popup.js
- Popup modules can import from `../background/constants.js` (both are ES modules in the same extension directory)

## Chat Module Pattern

- `popup/chat/` — agent chat panel (conversation loop, rendering, state, tools, system prompt)
- `chat.js` — conversation loop (`sendMessage`), event wiring, `initChat()`. Imports from `state.js` and `render.js`
- `state.js` — all chat state (`messages`, `busy`, `aborted`, `contextLimit`), persistence (`saveHistory`/`loadHistory`/`clearHistory`), `setBusy`/`stopAgent`, LLM timeout wrapper, friendly error mapping, a11y updates
- `render.js` — all DOM rendering: `renderMessage`, `renderToolCall`, `renderError`, `renderCompaction`, `renderThinking`/`updateThinking`/`removeThinking`, `appendAndScroll`, `renderAllMessages`
- `system.js` — builds system prompt from `getDomain()` + `getGuidance()` (supports user overrides via `prompt.js`)
- `prompt.js` — Agent/Prompt tab switcher, custom domain/guidance editing, save/defaults, persisted in `chrome.storage.local`
- `compaction.js` — LLM-powered compaction (summarizes older messages when context exceeds 75% threshold)
- `context.js` — tool result summarization + history pruning (fallback to compaction)
- `tools/` — one file per tool (11 tools) + `index.js` barrel with `TOOL_DEFS` and `executeTool`
- **State sharing**: `state.js` exports `let` bindings (ES module live bindings). `chat.js` reads them directly; mutation is via `.push()` on the array or `setMessages()` for replacement. Do NOT create a second `messages` variable in `chat.js`

## Agent-First Accessibility

The popup DOM is designed for agent consumption via Playwright accessibility snapshots (`browser_snapshot`) and CDP `Accessibility.getFullAXTree`. Every UI element must be self-describing to an agent reading the a11y tree.

### Semantic HTML (structure)
- `<header>`, `<nav>`, `<main>`, `<section>` landmarks — agents see page structure
- `<h1>`-`<h3>` heading hierarchy — section labels are `<h2>`, plot fields are `<h3>`
- `<details>`/`<summary>` for collapsible sections (story cards) — native `[expanded]` state
- `<dialog>` with `.showModal()`/`.close()` for overlays — native `[modal]` state, focus trapping, Escape to close
- `hidden` attribute (not `style="display:none"`) — properly excluded from a11y tree
- `role="status"` on connection indicators, `role="alert"` on errors, `role="log"` on progress

### Agent Metadata (from `helpers.js`)
- **`describe(el, meta)`** — sets `aria-description` from key:value pairs. Agents see `[description="shortId:abc, cards:3"]`
- **`agentRole(el, role)`** — sets `aria-roledescription`. Agents see `branch` instead of `listitem`, `story-card` instead of `group`
- **`agentHint(parent, text)`** — creates visually-hidden `.agent-hint` text block. Invisible to humans, visible in a11y tree. Contains workflow instructions and context summaries

### Labeling Rules
- Every interactive element needs `aria-label` with **action + context**: "Clone Path of Embers" not just "Clone"
- Decorative elements (chevrons, icons, dots) get `aria-hidden="true"` — removes noise from snapshots
- `document.body` gets `describe()` with current view state: `view:parent-list, scenario:CTU, branches:4`
- Branch items get `describe()` with shortId. Story cards get `describe()` with type and id
- Each major view gets `agentHint()` summarizing available actions and how to perform them

### Testing Paths (both read the same a11y tree)
- **Primary**: Playwright MCP — `browser_tabs select` to focus popup tab, `browser_snapshot` for tree, `browser_click` for interaction
- **Fallback**: `dev/test.mjs` — CDP `Runtime.evaluate` with `window.ctx`, `window.openClonePanel()`, etc.
- **Future**: `dev/test.mjs snap` using CDP `Accessibility.getFullAXTree` for the same tree without Playwright

## Code Style

- Small code files. **Ideal range: 150–250 lines.** Split before exceeding ~300 lines
- No `state.js` in background/ — use `storage.js` helpers instead (popup uses `ctx` in helpers.js for transient state)
- Constants are never duplicated across files
- `PLOT_FIELDS` in `constants.js` is the canonical plot component list — shared between popup and background
- Stubs are marked with `// TODO:` and their milestone number
- All GraphQL queries/mutations are marked `// TODO: UNTESTED` until verified against live traffic

## Optimistic UI Pattern

AID's API has read-after-write inconsistency — `GET_SCENARIO` called immediately after a mutation (delete, clone) returns stale data. **Never re-fetch to update the branch list after a mutation.** Instead:

- **Delete**: Filter the deleted `shortId` out of `ctx.branches` locally. In parent view, call `rebuildBranchList()`. In drill-down view, remove the `<li>` via `[data-shortid]` selector
- **Clone (new branch)**: Dispatch `CustomEvent("mca:branch-added", { detail: { shortId, title } })`. popup.js handles it: pushes to `ctx.branches` in parent view, or appends `makeBranchItem()` to `$("branch-children-list")` in drill-down view
- **Clone (to existing target)**: No branch list change — just close the panel
- **Branch items** always have `data-shortid` attribute for direct DOM lookup by the optimistic handlers
- **Future mutations** (rename, reorder) should follow the same pattern: mutate server, update local state/DOM, skip re-fetch

---
> Source: [LewdLeah/Multiple-Choice-Assistant](https://github.com/LewdLeah/Multiple-Choice-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
