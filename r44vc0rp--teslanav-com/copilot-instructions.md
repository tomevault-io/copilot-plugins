## teslanav-com

> TeslaNav is a navigation web app optimized for Tesla's in-car browser. Built with Next.js 16 (App Router), React 19, TypeScript 5, and Tailwind CSS 4.

# AGENTS.md - TeslaNav Codebase Guidelines

TeslaNav is a navigation web app optimized for Tesla's in-car browser. Built with Next.js 16 (App Router), React 19, TypeScript 5, and Tailwind CSS 4.

## Build, Lint & Test Commands

```bash
bun install              # Install dependencies (package manager: Bun)
bun run dev              # Start dev server (http://localhost:3000)
bun run build            # Production build
bun run start            # Start production server
bun run lint             # Run ESLint across project
bunx eslint <file>       # Lint a specific file
bunx eslint --fix <file> # Auto-fix lint issues in a file
bunx tsc --noEmit        # Type check without emitting output
bun run scripts/test-waze-bounds.ts  # Manual Waze API bounds test (CLI utility)
```

**No test framework is configured.** There are no unit/integration tests. Validation is done via TypeScript strict mode and manual browser testing. Add `?dev=true` to the URL to enable debug overlays.

## Project Structure

```
app/                    # Next.js App Router - pages and API routes
  api/                  # Server-side API routes (route.ts per endpoint)
  admin/                # Admin dashboard (usage stats)
  record/               # GPX track recording page
  view/                 # GPX track playback page
components/             # React components (ui/ for shadcn/ui primitives)
hooks/                  # Custom React hooks (use* prefix, camelCase.ts files)
lib/                    # Shared server/client utilities
  redis.ts              # Upstash Redis client, cache keys, TTLs, rate limits, API tracking
  utils.ts              # cn() = clsx + tailwind-merge
  gpx.ts                # GPX generation, parsing, interpolation
  posthog-server.ts     # Server-side PostHog singleton
types/                  # TypeScript type definitions (one file per domain)
public/                 # Static assets (icons/, cars/, sw.js service worker)
scripts/                # Developer utility scripts (not part of the app)
```

## Code Style Guidelines

### Import Organization

```typescript
"use client";  // 1. Directive first (if needed)

import { useState, useCallback } from "react";  // 2. React
import mapboxgl from "mapbox-gl";               // 3. External packages
import { cn } from "@/lib/utils";               // 4. Internal path-aliased (@/)
import { LocalComp } from "./LocalComp";        // 5. Relative imports
import type { MyType } from "@/types/foo";      // 6. Type-only imports last
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `NavigateSearch`, `SettingsModal` |
| Component files | PascalCase.tsx | `Map.tsx`, `FeedbackModal.tsx` |
| Hooks | camelCase `use` prefix | `useGeolocation`, `useWazeAlerts` |
| Hook files | camelCase.ts | `useGeolocation.ts` |
| Types/Interfaces | PascalCase | `WazeAlert`, `RouteData` |
| Type files | camelCase.ts | `waze.ts`, `route.ts` |
| API routes | route.ts | `app/api/directions/route.ts` |
| Constants | UPPER_SNAKE_CASE | `CACHE_TTL`, `RATE_LIMITS` |
| Functions/variables | camelCase | `fetchDirections`, `handleClick` |
| localStorage keys | `teslanav-` prefix | `teslanav-theme`, `teslanav-follow-mode` |

### TypeScript Guidelines

- **Strict mode is enabled** — all code must pass `bunx tsc --noEmit`
- Use `interface` for object shapes that may be extended; `type` for unions and computed types
- Prefer explicit return types on all exported functions
- Use the `type` keyword for type-only imports: `import type { Foo } from "..."`
- No `any` — use `unknown` with narrowing or proper generic types
- Zod (`zod`) is available for runtime validation of external API responses

### Component Patterns

```typescript
"use client";

import { useState, useCallback, forwardRef } from "react";

interface MyComponentProps {
  required: string;
  optional?: number;
  onAction?: (value: string) => void;
}

// Named exports for components (not default exports, except page.tsx files)
export const MyComponent = forwardRef<HTMLDivElement, MyComponentProps>(
  function MyComponent({ required, optional = 0, onAction }, ref) {
    const handleClick = useCallback(() => {
      onAction?.(required);
    }, [onAction, required]);

    return <div ref={ref} onClick={handleClick}>{/* content */}</div>;
  }
);
```

- Page files (`app/**/page.tsx`) use `export default function`
- All other components use named exports: `export const Foo = ...` or `export function Foo`
- Wrap callbacks in `useCallback` with proper dependency arrays
- Use `useRef` for animation frames, timers, DOM elements, and mutable counters that should not trigger re-renders
- Private helper sub-components (not exported) may live at the bottom of a file (e.g., `Toggle`, `CloseIcon`)
- Prefer inline SVG for one-off icons rather than importing from lucide-react

### State Management

- `useState` with lazy initializer for localStorage-persisted preferences:
  ```typescript
  const [theme, setTheme] = useState(() => localStorage.getItem("teslanav-theme") ?? "dark");
  ```
- `useRef` for values that should not trigger re-renders (animation state, timers, previous values)
- Dark mode is passed as a prop (`isDarkMode: boolean`), not via context or CSS class
- Server-side state uses Redis (`@/lib/redis`) — see cache keys defined in `CACHE_KEYS`

### API Route Patterns

All API routes follow: **validate params → check Redis cache → check rate limit → fetch upstream → cache result → return response**

```typescript
import { NextRequest, NextResponse } from "next/server";
import { redis, CACHE_KEYS, CACHE_TTL } from "@/lib/redis";

export async function GET(request: NextRequest) {
  const param = request.nextUrl.searchParams.get("param");
  if (!param) {
    return NextResponse.json({ error: "Missing required parameter" }, { status: 400 });
  }

  // Cache check
  const cached = await redis.get<DataType>(CACHE_KEYS.myKey(param));
  if (cached) return NextResponse.json(cached, { headers: { "X-Cache": "HIT" } });

  try {
    const data = await fetchUpstream(param);
    await redis.set(CACHE_KEYS.myKey(param), data, { ex: CACHE_TTL.MY_TTL });
    // Fire-and-forget usage tracking
    trackApiUsage("my_api").catch(console.error);
    return NextResponse.json(data, { headers: { "X-Cache": "MISS" } });
  } catch (error) {
    console.error("[MyAPI] fetch failed:", error);
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}
```

- Return stale cache rather than hard-failing when upstream is unavailable
- Log errors with `[RouteName]` prefix: `console.error("[Waze]", error)`
- Consistent error shape: `{ error: "message" }` with appropriate HTTP status
- Include `X-Cache: HIT | MISS | STALE` header on cacheable responses

### Error Handling

- Wrap all async operations in `try/catch`
- Fire-and-forget side effects (analytics, usage tracking): `doThing().catch(console.error)`
- Rate-limited endpoints return `429` with an empty-data body (graceful degradation, not hard failure)
- Stale Redis cache is preferred over an upstream error response

### Styling

- Tailwind CSS v4 (CSS-first config via `app/globals.css` — no `tailwind.config.*` file)
- Use `cn()` from `@/lib/utils` to merge conditional classes:
  ```typescript
  <div className={cn("base-classes", isActive && "active-classes", className)} />
  ```
- CSS custom properties are defined with `oklch()` color values in `globals.css`
- Dark mode: satellite mode forces dark UI via `effectiveDarkMode = isDarkMode || useSatellite`
- shadcn/ui components live in `components/ui/` and use CVA (`class-variance-authority`) for variants

### Hooks Pattern

```typescript
// hooks/useMyFeature.ts
export function useMyFeature(param: string) {
  const [data, setData] = useState<DataType | null>(null);
  const timerRef = useRef<NodeJS.Timeout | null>(null);

  useEffect(() => {
    // setup
    return () => {
      // cleanup: clear timers, cancel animation frames, abort fetches
      if (timerRef.current) clearTimeout(timerRef.current);
    };
  }, [param]);

  return { data };
}
```

- Always clean up timers, `requestAnimationFrame` handles, and `watchPosition` IDs in effect cleanup
- Debounce map-dependent fetches (250–300ms) to avoid over-fetching on pan/zoom
- Use `useRef` to store the last fetch bounds; only re-fetch when movement exceeds a threshold

## Key Libraries

| Library | Usage |
|---------|-------|
| `mapbox-gl` | Map rendering and Directions API |
| `@upstash/redis` | Server-side caching and rate limiting |
| `@vercel/blob` | GPX file storage |
| `zod` | Runtime validation of external API responses |
| `posthog-js` / `posthog-node` | Client + server analytics |
| `@radix-ui/*` | Accessible UI primitives (via shadcn/ui) |
| `lucide-react` | Icon library |
| `class-variance-authority` | Component variant styling |
| `clsx` + `tailwind-merge` | Conditional class merging (`cn()`) |

## Environment Variables

```bash
NEXT_PUBLIC_MAPBOX_TOKEN        # Mapbox GL JS + Directions API token
UPSTASH_REDIS_REST_URL          # Upstash Redis endpoint
UPSTASH_REDIS_REST_TOKEN        # Upstash Redis auth token
NEXT_PUBLIC_POSTHOG_KEY         # PostHog project API key
NEXT_PUBLIC_POSTHOG_HOST        # PostHog host (e.g. https://us.posthog.com)
BLOB_READ_WRITE_TOKEN           # Vercel Blob storage token
LOCATIONIQ_API_KEY              # LocationIQ geocoding API key
INBOUND_API_KEY                 # Inbound email API key (feedback + alerts)
ADMIN_API_KEY                   # Bearer token for /api/admin/* routes
```

## Special Notes

1. **Tesla Browser**: Optimized for Tesla's in-car Chromium browser. Avoid hover-only interactions. Test with touch events. The service worker (`public/sw.js`) caches map tiles for offline use.
2. **Dev Mode**: Add `?dev=true` to the URL to enable tile bounds debug overlay and verbose logging.
3. **Rate Limiting**: All external API calls (Waze, OSM, LocationIQ) are rate-limited via Redis. Constants are in `lib/redis.ts` (`RATE_LIMITS`, `API_LIMITS`).
4. **Path Alias**: Always use `@/` for imports from project root (configured in `tsconfig.json`).
5. **No Tailwind config file**: Tailwind v4 is configured entirely in `app/globals.css`. Do not create `tailwind.config.*`.
6. **Tile caching**: Map tiles are proxied through `/api/tiles` and cached in Vercel Blob (15-day TTL). Do not bypass this proxy.
7. **API usage tracking**: Call `trackApiUsage("api_name")` fire-and-forget in routes that consume metered external APIs. Thresholds and alerts are managed in `lib/redis.ts`.

---
> Source: [R44VC0RP/teslanav.com](https://github.com/R44VC0RP/teslanav.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
