## customproxmoxwebui

> This is a Proxmox VE management UI. Two services live in this repo:

# Coding standards for this repo

This is a Proxmox VE management UI. Two services live in this repo:

- `backend/` — Rust (axum) HTTPS proxy in front of the Proxmox REST API
- `frontend/` — React + TypeScript + Vite + Tailwind + shadcn/ui

The full project brief is in `docs/project-impl.md`. Read it before doing
non-trivial work — it covers architecture, design language, the Proxmox API
quirks, and the roadmap by phase.

## Hard rules

- TypeScript strict mode. No `any` without a `// reason: ...` comment.
- All API responses typed end-to-end. Backend types are mirrored in
  `frontend/src/api/*.ts`.
- No `console.log` in shipped code. Backend uses `tracing`. Frontend uses
  `src/lib/log.ts`.
- Never log API tokens or full Proxmox request bodies. The Proxmox token
  never reaches the browser.
- Cookies: `HttpOnly`, `SameSite=Strict`, `Secure` when `SESSION_COOKIE_SECURE=true`.
- Validate inputs at the route layer (vmid is a u32, etc.).
- Don't proxy arbitrary Proxmox endpoints. Each backend route is an explicit,
  audited mapping.
- No credits in git commit messages

## Design tokens

`frontend/src/styles/globals.css` defines semantic CSS variables
(`--bg-base`, `--bg-elevated`, `--text-primary`, `--accent`, ...). Tailwind
is configured to expose them as utility classes (`bg-base`, `text-primary`,
etc.). Do **not** use `slate-*` / `gray-*` / `zinc-*` directly in components.
If you need a color, it lives in the token system.

Status indicators are 6px colored dots, not filled badges.
Resource meters are 4-6px thin bars + numeric percentage to the right.
Cards: 8px radius. Buttons: 6px. Inputs/badges: 4px.

## Loading / empty / error states

Every list and detail view has all three. Skeletons match the loaded layout —
no generic spinners. Empty states are designed (icon + label + primary action),
not blank.

## Polling

TanStack Query handles all server state. Polling intervals (set per-query,
not as a global default):

- Cluster status: 10s
- VM/LXC list: 5s
- VM/LXC detail: 3s
- rrd graphs: 30s

Query keys live in `frontend/src/lib/query-keys.ts`. Add new ones there.

## Backend conventions

- Each route handler is small. Validation → call `ProxmoxClient` →
  transform → return.
- Errors: return `AppError`. Its `IntoResponse` impl produces the standard
  JSON error envelope (`{"error": {"code", "message", "details"}}`).
- The Proxmox API wraps everything in `{"data": ...}`. The client unwraps
  this; route handlers see clean values.
- Most VM/LXC routes need to know which node hosts the resource. Use
  `ProxmoxClient::find_node_for(kind, vmid)` which queries
  `/cluster/resources` (cached 30s).

## Frontend conventions

- File-based routing under `src/routes/`. Each leaf route is a component;
  data is loaded with TanStack Query inside the component, not in
  `loader`s, so we can use polling and Suspense fallbacks consistently.
- Mutations: optimistic update on the relevant query, then invalidate on
  settle. Show a toast on error.
- Format helpers (`bytes`, `pct`, `duration`, `relativeTime`) live in
  `src/lib/format.ts`. Don't reimplement them inline.

---
> Source: [0xntpower/CustomProxmoxWebUI](https://github.com/0xntpower/CustomProxmoxWebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
