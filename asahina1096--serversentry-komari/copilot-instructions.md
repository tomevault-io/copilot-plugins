## serversentry-komari

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ServerSentry is a Next.js 15 + React 19 monitoring dashboard for ServerStatus-Rust. It provides real-time monitoring of server metrics including CPU, memory, disk usage, and network traffic with a modern, responsive UI designed for optimal performance.

## Development Commands

- `bun run dev` - Start development server with Turbopack
- `bun run build` - Build production version
- `bun run start` - Start production server
- `bun run lint` - Run ESLint
- `bun install` - Install dependencies (Bun is the preferred package manager)

## Architecture

### Core Technologies
- **Next.js 15** with App Router and standalone output
- **React 19** with Strict Mode enabled
- **TypeScript** targeting ES2022 for modern builds
- **Tailwind CSS v4** with custom font (HarmonyOS Sans SC) and native animations
- **TanStack Query v5** for data fetching, caching, and polling
- **Native Fetch API** (no axios) with Komari RPC2 backend integration

### Project Structure

```
src/
├── app/                 # Next.js App Router pages
├── components/          # React components
│   ├── dashboard/       # Dashboard-specific components
│   ├── server/          # Server card components
│   └── ui/              # Reusable UI components
├── hooks/               # Custom React hooks
├── lib/                 # Utilities and configuration
├── types/               # Shared TypeScript interfaces
└── styles/              # CSS files
```

### Key Configuration Files

- `src/lib/config.ts` - Global app configuration including API URLs, refresh interval, and site metadata
- `src/lib/api.ts` - Frontend helpers (response typing, region grouping utilities)
- `src/lib/rpc2.ts` - Komari RPC2 client and typed wrappers
- `src/lib/response.ts` - Helper to standardize API route JSON responses and cache policy
- `src/types/server.ts` - Shared server interface for UI consumption
- `next.config.ts` - Next.js configuration with standalone output

### Data Flow

1. **API Integration**: The frontend consumes local Next.js routes (`/api/servers`, `/api/public`, `/api/version`).
   - `/api/servers` aggregates Komari data primarily via RPC2 (`common:getNodes`, `common:getNodesLatestStatus`) and supplements uptime through the traditional REST endpoint (`/api/recent/{uuid}`) when required.
2. **Query Management**: TanStack Query polls `/api/servers` at `config.refresh.serversMs` (default 2s) with window-focus refetch disabled to reduce churn.
3. **Data Processing**: API routes normalize Komari responses (network totals, memory/disk units, virtualization labels) before sending data to the UI.
4. **Type Safety**: Shared TypeScript interfaces live in `src/types/server.ts`, keeping hooks and components aligned with backend payloads.

### UI Design Philosophy

- **Performance-First**: No heavy animation libraries (framer-motion removed)
- **Lightweight Animations**: CSS-only transitions and hover effects
- **Responsive Design**: Mobile-first approach with Tailwind CSS
- **Accessibility**: Proper semantic HTML and reduced motion support

### Performance Optimizations

- **Component Lazy Loading**: Large components (ServerList, RegionSelect, RegionGroupView) are lazy-loaded using React.lazy()
- **Virtualized Lists**: Server collections over 50 items use a lazy-loaded optimized list component to reduce render cost
- **Query Optimization**: TanStack Query configured with disabled window focus refetch and tight polling interval
- **Data Selection**: Hooks return pre-sorted/grouped data to minimize downstream processing
- **Turbopack**: Fast development builds with Turbopack
- **CSS-Only Animations**: No JavaScript animation libraries - all transitions via Tailwind CSS classes
- **Modern Browser Target**: Next.js build targets modern browsers (ES2022) and omits legacy polyfills
- **macOS Scroll Optimization**: Specialized CSS classes for smooth scrolling on macOS
- **GPU Acceleration**: Hardware acceleration for all interactive elements
- **Layout Containment**: CSS containment to reduce reflow and repaint
- **Optimized Scrollbars**: Thinner, GPU-accelerated scrollbars
- **Background Rendering**: Optimized backdrop-filter and background effects

### Theme System

- Supports light/dark mode toggle via next-themes
- Configurable background system with gradient animations
- Custom theme selector component
- CSS variables for consistent theming
- Default theme: light, system theme detection disabled

### Key Features

- **Dashboard Stats**: Aggregated view of total/online servers, uptime, and traffic
- **Region Filtering**: Filter servers by location (groups servers by `location` field)
- **Region Grouping**: Displays servers grouped by region with collapsible sections
- **Real-time Monitoring**: Auto-refreshing server metrics (CPU, memory, disk, network)
- **Network Speed Display**: Live upload/download speeds with formatted units
- **Status Indicators**: Visual IPv4/IPv6 connectivity status badges
- **Loading States**: Skeleton screens and optimistic UI updates

## Environment Variables

- `KOMARI_BASE_URL` - Komari backend base URL
- `KOMARI_API_KEY` - Optional Komari API key (Bearer), enables full data where required

## Deployment

The app is configured for standalone deployment with:
- **Standalone Output**: Next.js standalone mode for minimal Docker images
- **Built-in API Proxy**: Next.js API routes handle Komari RPC2 calls, REST fallbacks, and CORS
- **Production Build**: Console logs removed in production builds
- **Environment**: Requires `KOMARI_BASE_URL` (and optional `KOMARI_API_KEY`) for backend access

### Deployment Options

1. **Standalone Node.js**: Deploy `.next/standalone` directory with Node.js 18+
2. **Nginx Reverse Proxy**: Proxy frontend on port 3000, ensure `KOMARI_BASE_URL` is configured
3. **Replace Built-in Theme**: Export and copy to ServerStatus-Rust `web/` directory (see README.md)

## Component Patterns

- **Component Organization**:
  - `components/dashboard/` - Dashboard-specific components
  - `components/server/` - Server card and detail components
  - `components/ui/` - Reusable UI primitives
  - Export components via index.ts barrel files
- **Lazy Loading**: Large components should use React.lazy() with Suspense fallbacks
  ```tsx
  const ServerList = lazy(() => import('@/components/server-list').then(m => ({ default: m.ServerList })));
  ```
- **Virtualized Rendering**: `VirtualizedServerList` is loaded on demand for large server sets (50+) to minimize DOM nodes.
- **Styling**:
  - Use Tailwind CSS classes exclusively (no CSS-in-JS)
  - Apply performance optimization classes for macOS:
    - `gpu-accelerated` - Hardware acceleration
    - `card-optimized` - Card components
    - `grid-optimized` - Grid layouts
    - `macos-scroll-optimized` - Smooth scrolling
  - Use `transition-*` classes for hover effects
- **Data Fetching**:
  - Use custom hooks (`use-servers.ts`, `use-region-data.ts`)
  - Follow TanStack Query patterns with select for data transformation
  - TypeScript interfaces defined in `src/types/server.ts`
- **Formatting Utilities**: Available in `src/lib/utils.ts`
  - `formatBytes(bytes, decimals)` - Format storage/memory sizes
  - `formatSpeed(bytes, decimals)` - Format network speeds
  - `formatPercent(value, total)` - Calculate percentages
  - `groupServersByRegion(servers)` - Group servers by location (`src/lib/api.ts`)
  - `getUniqueRegions(servers)` - Extract unique region list (`src/lib/api.ts`)
  - All use native `Intl.NumberFormat` for localization

---
> Source: [Asahina1096/ServerSentry-Komari](https://github.com/Asahina1096/ServerSentry-Komari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
