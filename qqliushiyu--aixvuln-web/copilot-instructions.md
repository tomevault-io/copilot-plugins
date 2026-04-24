## aixvuln-web

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

AIxVuln Web is the frontend for an AI-powered code security audit platform. Users upload source code archives (.zip/.tar.gz), and the backend runs vulnerability analysis in Docker containers. Results stream back via WebSocket in real-time.

**Backend repository:** [AIxVuln](https://github.com/m4xxxxx/AIxVuln) (separate project, not in this repo)

## Build & Development Commands

```bash
npm run dev      # Start development server (http://localhost:3000)
npm run build    # Production build
npm run start    # Run production build
npm run lint     # ESLint check
```

## Environment Variables

Copy `.env.local.example` to `.env.local` and configure:

| Variable | Description | Example |
|----------|-------------|---------|
| `BACKEND_URL` | Backend API address (server-side only) | `http://localhost:8080` |
| `NEXT_PUBLIC_WS_BASE` | WebSocket direct connection URL (client-side) | `ws://localhost:8080` |

## Architecture

### Tech Stack
- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 4
- **Components:** shadcn/ui + Radix UI

### Key Directories

```
src/
├── app/                        # Next.js App Router
│   ├── page.tsx                # Project list (home page)
│   ├── projects/[name]/page.tsx # Project detail page
│   └── api/                    # API route handlers
│       ├── proxy/[...path]/    # Backend API proxy (handles auth)
│       ├── upload/             # File upload endpoint
│       └── logout/             # Authentication logout
├── components/
│   ├── audit/                  # Business components
│   └── ui/                     # shadcn base components
├── hooks/
│   └── useWebSocket.ts         # WebSocket hook for real-time updates
├── lib/
│   ├── api.ts                  # API service layer
│   └── types.ts                # TypeScript interfaces
└── middleware.ts               # Security headers (CSP, HSTS, etc.)
```

### Data Flow

1. **API Requests:** All backend calls go through `/api/proxy/[...path]` which forwards requests to `BACKEND_URL` and handles Basic Auth.

2. **File Uploads:** Large file uploads (up to 500MB) use `/api/upload` endpoint with XHR for progress tracking.

3. **Real-time Updates:** WebSocket connects directly to backend (`NEXT_PUBLIC_WS_BASE`) - Next.js doesn't proxy WebSocket. The `useWebSocket` hook manages connection lifecycle and dispatches events by type:
   - `VulnAdd` / `VulnStatus` - Vulnerability updates
   - `ContainerAdd` / `ContainerRemove` - Docker container status
   - `ReportAdd` - New audit reports
   - `EnvInfo` - Environment/database info
   - `ProjectStatus` - Project state changes
   - `string` - Event log messages

### Key Patterns

- **Authentication:** Browser-native Basic Auth. The proxy forwards `Authorization` headers to backend. Logout returns 401 to clear cached credentials.

- **Path alias:** Use `@/*` to import from `src/` (e.g., `@/lib/api`, `@/components/ui/button`)

- **Component exports:** Business components in `src/components/audit/` are re-exported from `index.ts`

### Important Types (src/lib/types.ts)

- `ProjectDetail` - Full project state including vulns, containers, reports
- `VulnStruct` - Vulnerability info (type, file, confidence, status, etc.)
- `ContainerStruct` - Docker container info (id, IP, image, ports)
- `WSMessage` - WebSocket message envelope with `type` and `data`

## Notes

- No test framework is currently configured
- The `ecosystem.config.js` is a PM2 config template (not actively used)
- Security middleware in `src/middleware.ts` adds CSP, X-Frame-Options, HSTS (production only), etc.

---
> Source: [qqliushiyu/AIxVuln_Web](https://github.com/qqliushiyu/AIxVuln_Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
