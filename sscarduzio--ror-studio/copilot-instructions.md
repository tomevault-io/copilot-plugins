## ror-studio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Vite dev server on port 4500
npm run build      # TypeScript check + Vite production build (tsc -b && vite build)
npm run lint       # ESLint
npm run knip       # Dead code detection (unused files, exports, dependencies)
npm run check      # All three above in sequence (tsc + eslint + knip)
npx vite preview --port 4501  # Serve production build (respects base path)
```

No test runner is configured. There are no test files.

**CI pipeline** (`.github/workflows/deploy.yml`): On every push/PR to `main`, runs `tsc`, `eslint`, and `knip` as a gate before build. Build failures, lint errors, or dead code will block deployment to GitHub Pages.

## What is ROR Studio

ROR Studio is a **client-side-only** companion editor for [ReadOnlyREST](https://readonlyrest.com), the security plugin for Elasticsearch and Kibana. It helps users build, validate, and export `readonlyrest.yml` configuration files through a visual interface — supporting all ReadOnlyREST editions (Free, PRO, Enterprise).

**Privacy by design**: everything runs in the browser with zero backend and zero external network requests. Your security settings never leave your machine. The app works fully air-gapped — all assets including the Monaco code editor are bundled locally.

## Architecture

Single-page React 19 app. Vite build. No backend, no API calls, no telemetry.

### State Management

**Zustand store** (`src/store/editor-store.ts`) is the single source of truth:
- `config: RorConfig` — the ACL configuration being edited
- `edition: 'free' | 'pro' | 'enterprise'` — controls which rule types are available; auto-upgrades when user adds a higher-tier rule or loads a template
- `activeTab: TabId` — current sidebar tab
- `aclViewMode: 'graph' | 'form' | 'code'` — ACL tab view mode (persisted)
- `yamlDock: 'bottom' | 'right'` — YAML preview panel position
- Undo/redo via `_past`/`_future` arrays (limit 100), with `pushHistory()` before mutations
- `setConfig(config)` pushes history; `setConfigSilent(config)` does not (used for initial restore and debounced Code editor updates to avoid flooding the undo stack)
- Block CRUD: `addBlock`, `updateBlock`, `removeBlock`, `reorderBlocks`, `duplicateBlock`
- User CRUD: `addUser`, `updateUser`, `removeUser`
- Generic nested field updates via `updateConfigField(path, value)` using `setNestedField()`

**Persistence** (`src/store/persistence.ts`): auto-saves to localStorage with 500ms debounce. All enum values (edition, tab, dock, view mode) are validated against known sets on load — corrupted localStorage falls back to safe defaults. Silent failure on quota exceeded.

### ACL Tab — Three View Modes

The ACL tab (`src/components/acl-flow/AclTab.tsx`) hosts three switchable modes behind a `ViewModeToggle`:

- **Graph** (`AclFlowGraph.tsx`) — G6 flow diagram with node styles in `graph-styles.ts`
- **Form** (`AccessControlTab`) — block card editor with drag-to-reorder
- **Code** (`AclCodeEditor.tsx`) — read-write Monaco YAML editor with its own validation pipeline

The view mode toggle (`ViewModeToggle.tsx`) is a generic reusable segmented control with sliding pill animation. Mode definitions in `acl-view-modes.ts` — adding a 4th mode = add one entry to the array.

### Code Editor Validation

`AclCodeEditor` runs its own 3-layer validation directly on the raw editor text (independent of the store-based `useValidation` hook):

1. **YAML syntax** — `js-yaml.load()` catches malformed YAML with line/column from `YAMLException.mark`
2. **ROR config parse** — `yamlToConfig()` catches structural issues
3. **Full ROR semantic** — `validateConfig()` runs all domain rules, mapped to YAML lines via `buildYamlLineMap()` + `resolveFieldToLine()`

Results are shown as Monaco squiggle markers + a clickable error summary bar above the editor.

Important: The Code editor uses `setConfigSilent` for debounced updates (not `setConfig`) to avoid flooding the undo stack. It pushes one undo snapshot on first edit via `hasSnapshotRef`, then uses silent updates for subsequent parses. Disabled blocks are preserved from the current store when parsing (they're serialized as YAML comments and can't be recovered by `yamlToConfig`).

### Data Flow

```
User edits form → Zustand store mutation → React re-render
                                        → useValidation() (200ms debounce) → semantic issues
                                        → configToYaml() → YAML text
                                        → buildYamlLineMap() → line numbers for issues
                                        → resolveFieldToLine() → maps internal field paths to YAML lines
                                        → Monaco markers (error squiggles at exact lines)
```

### Key Domain Types (`src/schema/types.ts`)

- `RorConfig` — top-level config: ACL blocks, connectors (LDAP/JWT/ROR KBN/external auth/groups providers), users, SSL, audit, global settings
- `AccessControlBlock` — has `id`, `name`, `type` (allow/forbid), `enabled`, `rules: AclRule[]`
- `AclRule` — `{ type: RuleType, value: unknown }` — flattened to YAML keys (e.g., `indices: [...]`)
- `RuleType` — union of ~77 string literals covering auth, network, ES, HTTP, Kibana rules
- `UserDefinition` — username, auth method (one of auth_key_*, ldap_auth, jwt_auth, etc.), local groups, optional authorization

### YAML Serialization (`src/utils/yaml.ts`)

`configToYaml()` converts internal `RorConfig` to valid `readonlyrest.yml`. Key detail: ACL blocks store rules as `{type, value}` arrays internally but YAML flattens them as top-level keys. Disabled blocks are serialized as YAML comments. Uses `js-yaml` with `indent: 2, lineWidth: -1, noRefs: true`.

`yamlToConfig()` parses YAML back, handling the `readonlyrest:` wrapper, legacy key aliases, and preserving unrecognized keys in `_unrecognized`. Note: generates new `crypto.randomUUID()` for each block on every parse — block IDs are not stable across roundtrips.

### YAML Line Mapping (`src/utils/yaml-line-map.ts`)

`buildYamlLineMap()` parses YAML text line-by-line to produce a `Map<fieldPath, lineNumber>`. Paths use dotted notation without `readonlyrest.` prefix (e.g., `access_control_rules[0].name`).

Critical detail: internal validator field paths don't match YAML paths (rules are flattened in YAML). `resolveFieldToLine()` in both `useValidation.ts` and `AclCodeEditor.tsx` bridges this gap — e.g., `access_control_rules[0].rules[2]` resolves to the YAML line for the 2nd rule's type key within block 0.

### Validation

- **Semantic validator** (`src/validation/semantic-validator.ts`): validates ACL blocks (names, rules, auth conflicts, connector references, kibana_access), users (empty usernames, missing auth, duplicate names, connector refs), and SSL consistency. Disabled blocks (`!block.enabled`) are skipped.
- **`useValidation()` hook** (`src/hooks/useValidation.ts`): debounced validation producing `ValidationResult` with issues, `issuesByTab`, counts, `yamlText`, and line numbers. Runs synchronously on first call (when `result === EMPTY_RESULT`) to avoid empty YAML preview flash on page load.
- **`ValidationIssue`** includes `fix` (actionable guidance), `fieldId` (DOM element ID for scroll-to-fix), `line`/`endLine` (YAML line numbers)
- **`useFieldErrors(prefix)`** hook: filters issues by field path prefix with boundary checking (prevents `[0]` from matching `[01]`)
- **Monaco markers**: issues with line numbers become red/yellow squiggles in the Monaco YAML preview

### Monaco Editor Setup (`src/utils/monaco-ror.ts`)

Shared `setupRorYaml(monaco)` registers the `ror-yaml` monarch language (with ROR variable highlighting `@{...}`) and `ror-light` theme. Fully idempotent — safe to call on every editor mount. `buildValidationMarkers()` converts `ValidationIssue[]` to Monaco marker data.

Monaco is bundled locally via `import * as monaco from 'monaco-editor'` + `loader.config({ monaco })` in `main.tsx`. The editor worker uses Vite's `new URL()` import pattern. No CDN dependencies.

### Edition Tier Gating

Rule metadata in `src/schema/field-meta.ts` includes a `tier` field per rule type. `getAllRulesByCategory()` returns all rules regardless of edition (for the dropdown). `isRuleAboveEdition()` compares tiers. When a user adds a rule above their edition, `BlockCard` shows a confirmation dialog and auto-switches the edition on accept. The "Add Rule" dropdown filters out already-used rule types to prevent duplicates (YAML flattens rules as keys — duplicates cause silent data loss).

### Shared Hooks

- `useHasContent()` (`src/hooks/useHasContent.ts`) — whether the config has any meaningful content (blocks, users, or connectors). Used by App, Header, Sidebar to show/hide UI sections.
- `useConnectorNames(configKey)` (`src/hooks/useConnectorNames.ts`) — shared between RuleEditor and UsersGroupsTab for connector dropdown options
- `AuthKeyEditor` (`src/components/acl/AuthKeyEditor.tsx`) — hash-and-lock UX for auth_key_* rules, reused in Users tab via `UserAuthKeyEditor` adapter

### Layout

App uses `react-resizable-panels` with dockable YAML preview:
- **Bottom dock** (default): Vertical split — top panel (Sidebar + TabContainer + ErrorPanel), bottom panel (YamlPreview)
- **Right dock**: Horizontal split — left panel (Sidebar + TabContainer + ErrorPanel), right panel (YamlPreview)
- Dock position toggle buttons in YamlPreview header bar

ErrorPanel sits between the tab content area and the panel separator — always visible when errors/warnings exist.

The "Add Rule" dropdown in BlockCard uses `createPortal` to render at document.body level, escaping the panel's `overflow-hidden`.

### UI Components

Radix UI primitives in `src/components/ui/`. Styling via Tailwind 4 + CSS custom properties defined in `src/index.css` (the `@theme` block). Shared style constants in `src/components/shared-styles.ts`.

Rule metadata (labels, descriptions, tiers, value types, placeholders) lives in `src/schema/field-meta.ts` — `getRuleMeta(type)` and `getAllRulesByCategory()`.

### Graph Styles (`src/components/acl-flow/graph-styles.ts`)

Pure data-mapping functions for G6 node/edge visual styles. `getNodeStyle()` and `getEdgeStyle()` are used by `AclFlowGraph`. User-card nodes use innerHTML with HTML escaping (`esc()`) to prevent XSS from malicious YAML imports.

### Getting Started / Onboarding

- **Templates** (`src/components/templates/GettingStartedTab.tsx`): 6 realistic template configs with preview dialog (Monaco editor) before loading
- **Wizard** (`src/components/wizard/`): 5-step guided setup for first ACL block (Welcome → Name → Auth → Permissions → Review)

### Path Alias

`@/` resolves to `src/` (configured in both `vite.config.ts` and `tsconfig.app.json`).

### Build & Deployment

Vite base path is `/ror-studio/`. Production build outputs to `dist/`. Use `npx vite preview` (not `npx serve`) to test the production build locally — it correctly handles the base path. Monaco editor + worker + codicon font are fully bundled (no CDN).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sscarduzio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sscarduzio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
