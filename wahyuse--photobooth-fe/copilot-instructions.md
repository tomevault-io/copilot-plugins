## photobooth-fe

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---

# AGENTS.md — Monolith Snap Frontend

> AI agent guide for the **Monolith Snap** photobooth frontend.
> Stack: **Next.js 16 · React 19 · TypeScript · Tailwind CSS v4 · Konva**

---

## 1. Project Overview

**Monolith Snap** is a modern in-event photobooth web app. It lets guests:

1. Start a timed session → pick a photo grid layout → take photos in the booth
2. Edit/compose the photo strip in a canvas editor (Konva)
3. Preview and share/save the result

An **Admin Console** (`/admin`) lets operators manage:
- Sessions & event configuration
- Photo templates (with a full Konva-based layout editor)
- Theme (colors, fonts, radius — applied globally as CSS variables)
- Canvas sizes (print dimensions, DPI)
- Activity logs

---

## 2. Technology Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js | 16.2.6 |
| UI library | React / React DOM | 19.2.4 |
| Language | TypeScript | ^5 |
| Styling | Tailwind CSS v4 | ^4.0.0 |
| Canvas rendering | Konva + react-konva | ^10 / ^19 |
| Webcam capture | react-webcam | ^7 |
| GIF generation | gifshot | ^0.4.5 |
| Photo-to-canvas | html2canvas | ^1.4.1 |
| QR codes | qrcode.react | ^4.2.0 |
| ID generation | uuid | ^14 |

> **Tailwind v4** uses `@import "tailwindcss"` and `@theme { }` — there is no `tailwind.config.js`.
> Do **NOT** use the Tailwind v3 config file approach.

---

## 3. Project Structure

```
frontend/
├── app/                          # Next.js App Router pages
│   ├── layout.tsx                # Root layout — loads theme, fonts (Sora, Plus Jakarta Sans)
│   ├── page.tsx                  # Landing page — "MULAI" session start button
│   ├── select-grid/              # Step 1: choose photo grid (1x1, 1x3, 3x4 …)
│   ├── select-frame/             # Step 2: choose template/frame
│   ├── booth/                    # Step 3: webcam capture (SSR disabled)
│   ├── editor/                   # Step 4: canvas editor (Konva)
│   ├── gallery/                  # Result gallery / QR share
│   ├── admin/                    # Admin console (password-protected)
│   │   ├── page.tsx              # Main admin SPA with tab routing
│   │   └── components/           # Per-tab sub-components
│   │       ├── DashboardTab.tsx
│   │       ├── TemplatesTab.tsx  # Template CRUD + visual layout editor
│   │       ├── ThemeTab.tsx      # Live theme editor
│   │       ├── SessionConfigTab.tsx
│   │       ├── CanvasSizesTab.tsx
│   │       ├── ActivityTab.tsx
│   │       └── Sidebar.tsx / NavItem.tsx / StatCard.tsx / StatusBadge.tsx
│   └── api/
│       └── gallery/              # Next.js route handler (proxies to backend)
├── components/
│   ├── Booth/
│   │   └── BoothClient.tsx       # Webcam UI — SSR disabled, client-only
│   ├── Editor/
│   │   ├── EditorClient.tsx      # Main canvas editor (Konva) — very large file
│   │   ├── CanvasEditor.tsx      # Canvas rendering layer
│   │   ├── AdminLayoutEditor.tsx # Template layout editor (admin)
│   │   └── TemplatePicker.tsx
│   ├── Session/
│   │   └── SessionTimer.tsx      # Countdown timer overlay
│   └── ThemeProvider.tsx         # Injects CSS variables from active theme
├── lib/
│   ├── theme.ts                  # UITheme type, fetchActiveTheme, buildCssVariables
│   ├── templates.ts              # Template helpers
│   └── canvasSizes.ts            # CanvasSize type, dimension helpers
├── types/
│   └── gifshot.d.ts              # Manual type declaration for gifshot
├── app/globals.css               # Tailwind v4 @theme block + CSS variables
└── next.config.ts                # No x-powered-by, allow SVG images
```

---

## 4. Environment Variables

All env vars live in `.env.local`. **Never** commit this file.

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_API_URL` | Full base URL of the backend API (e.g. `http://localhost:4000`) |

Usage pattern in code:
```ts
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
fetch(`${apiUrl}/api/sessions/start`, { ... });
```

---

## 5. Theming System

The app uses a **fully dynamic CSS variable theme** fetched from the backend at runtime.

### Flow
1. `app/layout.tsx` calls `fetchActiveTheme()` (server-side, `cache: "no-store"`)
2. `ThemeProvider` receives `initialTheme` and injects a `<style>` tag with `--theme-*` variables
3. On client mount, `ThemeProvider` re-fetches and updates if theme changed
4. `globals.css` maps `--theme-*` → Tailwind `@theme` tokens (e.g. `--color-primary`)

### UITheme shape (`lib/theme.ts`)
```ts
interface UITheme {
  colorPrimary, colorPrimaryHover, colorSecondary,
  colorBackground, colorCard,
  colorText, colorTextMuted, colorBorder,
  colorError, colorSuccess,
  fontHeading, fontBody, fontMono,
  radiusBase, radiusLg, radiusXl
}
```

### Rules
- **Always use Tailwind semantic tokens** (`bg-primary`, `text-text-main`, `border-border-theme`) — never hard-code hex/rgb colors in components
- New color additions must go through `UITheme` → `buildCssVariables()` → `globals.css @theme` in that order
- Do not use `bg-pink-500` or other Tailwind palette colors directly; they won't follow the active theme

---

## 6. Session Management

Sessions are stored in `localStorage` and verified against the backend on each protected page.

| Key | Value |
|---|---|
| `pb_session_id` | Session UUID |
| `pb_session_expires` | ISO date string |
| `pb_admin_token` | Admin JWT (admin only) |

### Guard pattern (used in `/booth/page.tsx`)
```ts
useEffect(() => {
  const id = localStorage.getItem("pb_session_id");
  const expires = localStorage.getItem("pb_session_expires");
  if (!id || !expires || new Date(expires) <= new Date()) {
    // clear & redirect
    window.location.replace("/");
    return;
  }
  // verify with backend heartbeat
  fetch(`${NEXT_PUBLIC_API_URL}/api/sessions/${id}/heartbeat`, { method: "POST" })
    .then(r => r.json())
    .then(d => { if (!d.success || !d.active) window.location.replace("/"); });
}, []);
```

- Always verify with the backend heartbeat, not just local time
- Fallback to local check when backend is unreachable (graceful degradation)

---

## 7. Webcam & Canvas Rules

- `BoothClient` must **never** be SSR'd — always import via `dynamic(() => import(...), { ssr: false })`
- Webcam pages must render a static skeleton during SSR to avoid hydration mismatches
- Canvas editors use **Konva** (`react-konva`). Do not mix html2canvas and Konva rendering in the same layer
- GIF export is done via `gifshot` — types are manually declared in `types/gifshot.d.ts`

---

## 8. Admin Console

The Admin console at `/admin` is a **client-side SPA** with tab-based routing (no URL changes):

| Tab key | Component | Purpose |
|---|---|---|
| `sessions` | `DashboardTab` | Session list & search |
| `templates` | `TemplatesTab` | Template CRUD + layout editor |
| `sessions_list` | `SessionConfigTab` | Event config (name, dates, quota, timers) |
| `activity` | `ActivityTab` | Audit log |
| `canvas_sizes` | `CanvasSizesTab` | Print canvas size management |
| `theme` | `ThemeTab` | Live theme color/font editor |

Authentication is password-based: POST `/api/admin/login` → store JWT in `localStorage` as `pb_admin_token`.

---

## 9. Backend API Conventions

All communication is with the backend at `NEXT_PUBLIC_API_URL`.

| Method | Endpoint pattern | Purpose |
|---|---|---|
| POST | `/api/sessions/start` | Start new session |
| POST | `/api/sessions/:id/heartbeat` | Verify session is active |
| GET | `/api/sessions/history` | List sessions (admin) |
| GET | `/api/templates` | List templates |
| POST | `/api/templates` | Create template |
| GET/POST | `/api/admin/event/config` | Event configuration |
| GET/POST | `/api/admin/categories` | Template categories |
| GET | `/api/admin/theme` | Fetch theme list |
| POST | `/api/admin/login` | Admin authentication |

**Response shape** (consistent across all endpoints):
```json
{ "success": true, "data": { ... } }
{ "success": false, "message": "Error reason" }
```

Always check `data.success` before using `data.data`.

---

## 10. Styling Conventions

- Framework: **Tailwind CSS v4** — config is in `globals.css` via `@theme { }` block
- Utility classes are available globally once imported via `@import "tailwindcss"` in `globals.css`
- Component-level styles use **CSS Modules** (`ComponentName.module.css`) alongside Tailwind utilities
- Global component classes (`.btn`, `.btn-primary`, `.input`, `.label`) are defined in `globals.css` under `@layer components`
- Fonts loaded from Google Fonts: **Sora** (heading), **Plus Jakarta Sans** (body), **Space Grotesk** (display)

---

## 11. Common Commands

```powershell
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Lint
npm run lint
```

---

## 12. Agent Rules & Gotchas

1. **Read `node_modules/next/dist/docs/`** before using any Next.js API — this is Next.js 16 which has breaking changes from v13/14
2. **Tailwind v4** — there is no `tailwind.config.js`. Use `@theme {}` in `globals.css`. Do not add `content:[]` arrays
3. **Never hard-code theme colors** — always use the CSS variable tokens (`--color-primary`, etc.)
4. **SSR guard** — any component using `window`, `localStorage`, or `navigator` must be client-only (`"use client"`) or wrapped in `useEffect`
5. **Session expiry** — always verify with backend heartbeat, don't trust localStorage alone
6. **`EditorClient.tsx` is very large (~54 KB)** — read carefully before editing; state is complex
7. **`TemplatesTab.tsx` is very large (~61 KB)** — same caution applies
8. **`export const dynamic = "force-dynamic"`** is set on the root layout — all pages are dynamically rendered (no static caching)
9. **Konva nodes** must use `ref` access for imperative operations — do not directly mutate stage state
10. **GIF generation** is async and memory-intensive — do not block the UI thread; always use callbacks

---
> Source: [WahyuSe/photobooth_fe](https://github.com/WahyuSe/photobooth_fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
