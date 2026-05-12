## neomind

> - **Backend**: Rust (Axum, event-driven)

# NeoMind - Edge AI Platform for IoT

## Tech Stack
- **Backend**: Rust (Axum, event-driven)
- **Frontend**: React 18 + TypeScript + Tailwind CSS + Zustand
- **Desktop**: Tauri 2.x

## Development Commands

```bash
# Rust Backend (project root)
cargo build && cargo test && cargo run -p neomind-cli -- serve  # port: 9375

# Tauri Desktop (web/)
cd web && npm install && npm run tauri:dev

# Web Frontend (web/)
npm run dev && npm run build
```

## Project Structure

```
NeoMind/
├── crates/           # Rust workspace
│   ├── neomind-core/        # Core traits and types
│   ├── neomind-api/         # Web API server (Axum)
│   ├── neomind-agent/       # AI Agent with tool calling and LLM backends
│   ├── neomind-devices/     # Device management (MQTT)
│   ├── neomind-storage/     # Storage (redb)
│   ├── neomind-messages/    # Messaging system
│   ├── neomind-rules/       # Rule engine
│   ├── neomind-extension-sdk/     # Extension SDK
│   ├── neomind-extension-runner/  # Extension process isolation
│   └── neomind-cli/         # CLI tools
├── web/src/          # React frontend (components, pages, hooks, store, types)
├── docs/guides/      # User documentation (en/zh)
└── data/             # Runtime databases (telemetry.redb, sessions.redb, etc.)
```

## Key Rules

- **Ollama API**: Use `/api/chat` (native), NOT `/v1/chat/completions`
- **Tauri Environment**: API base is `http://localhost:9375/api`, WebSocket uses `ws://`
- **Time-series DB**: All metrics in `data/telemetry.redb`
- **DataSourceId Format**: `{type}:{id}:{field}` (e.g., `extension:weather:temp`)

## Code Conventions

- Rust: Follow standard Rust conventions, use `cargo fmt` and `cargo clippy`
- Frontend: ES modules, functional components, Zustand slices pattern
- Always run type checks after code changes

### Frontend Design & Component Standards

> **Full specification:** [`web/DESIGN_SPEC.md`](web/DESIGN_SPEC.md) — MUST read before any frontend UI work.

Key rules summarized:

- **Colors**: Only use design token classes (`text-success`, `bg-error-light`, `text-accent-orange`, etc.). NEVER use hardcoded Tailwind palette colors (`bg-blue-500`, `text-green-600`, etc.). Text on colored backgrounds uses `text-primary-foreground`.
- **Opacity limitation**: CSS variable-based colors do NOT support Tailwind `/` opacity modifier (e.g., `bg-primary/10` silently fails). Use pre-defined tokens (`bg-muted-30`, `bg-success-light`) or inline styles.
- **Loading States**: Page-level loading must use **skeleton screens** (`LoadingState variant="page"` or `ResponsiveTable` built-in). Spinner (`Loader2`) only for inline/button/dialog-level.
- **Pagination**: Default page size is **10**. Mobile uses infinite scroll via `hideOnMobile`.
- **Page Layout**: Use `PageLayout` with `PageTabsBar`/`PageTabsContent`. Content grows naturally; scroll handled by `PageLayout`.
- **Dialogs**: Use `UnifiedFormDialog` for form dialogs, `FullScreenDialog` for builders. Do NOT use raw `Dialog` directly.
- **UI Components**: Import from `@/components/ui/` (Button, Input, Select, Checkbox, Switch, Label, etc.). Do NOT use raw HTML form elements.
- **Status Colors**: Use `getStatusColorClass()` / `getStatusBgClass()` from `@/design-system/utils/format`.
- **Fetch Deduplication**: Store-level `fetchCache` (TTL 10s). Pattern: `shouldFetch` → `markFetching` → API call → `markFetched`. Invalidate on mutations. WebSocket events use optimistic updates.
- **Z-Index**: Popovers at `z-[200]`, full-screen dialogs at `z-[100]`/`z-[110]`, overlays at `z-50`.
- **i18n**: All user-visible text via `t()`. Never hardcode strings.
- **Portals**: All modals/popovers must use `getPortalRoot()` from `@/lib/portal`.

## Documentation

For detailed information, see:
- **API**: `docs/guides/en/14-api.md`
- **LLM**: `docs/guides/en/02-llm.md`
- **Agents**: `docs/guides/en/03-agent.md`
- **Devices**: `docs/guides/en/04-devices.md`
- **Storage**: `docs/guides/en/10-storage.md`
- **Extensions**: `docs/guides/en/extension-system.md`

---
> Source: [camthink-ai/NeoMind](https://github.com/camthink-ai/NeoMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
