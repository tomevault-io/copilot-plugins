## homepage-deferred-loading-pattern

> This rule defines the **mandatory** pattern for data fetching on the home page (and any public-facing landing page). All API / database calls MUST be deferred until **after** the browser has completed the initial paint of static content. This eliminates network request bursts that compete with rendering during hydration, producing a significantly faster perceived load — especially critical for AWS Amplify Lambda deployments where cold starts already add latency.

# Homepage Deferred Loading Pattern — Faster Initial Page Load on AWS Amplify

## Overview

This rule defines the **mandatory** pattern for data fetching on the home page (and any public-facing landing page). All API / database calls MUST be deferred until **after** the browser has completed the initial paint of static content. This eliminates network request bursts that compete with rendering during hydration, producing a significantly faster perceived load — especially critical for AWS Amplify Lambda deployments where cold starts already add latency.

## Problem Solved

- **Slow initial page load**: On first visit (cold start), multiple API calls fire simultaneously during React hydration, competing with the browser's paint and blocking the main thread with JSON parsing and state updates.
- **Network request burst**: 7+ concurrent API calls overwhelm the backend and the browser's connection limit (6 per origin in HTTP/1.1), queueing requests and delaying everything.
- **Static content blocked**: Users see a blank/loading page instead of immediately visible static sections (Services, About, Causes, etc.) because the main thread is busy processing API responses.

## Core Architecture

### Hook: `usePageReady` (`src/hooks/usePageReady.ts`)

Returns `true` after the browser has completed the initial paint cycle using nested `requestAnimationFrame`. This is the foundation for all deferred loading.

### Hook: `useDeferredFetch(delayMs)` (`src/hooks/usePageReady.ts`)

Returns `true` after `usePageReady()` + an additional configurable delay. Different delays stagger API calls so they don't fire simultaneously.

### Staggered Delay Tiers

| Priority | Delay | Component | Reason |
|----------|-------|-----------|--------|
| 0 (gate) | Page ready (~30ms) | `TenantSettingsProvider` | Determines section visibility; other sections depend on it |
| 1 (above fold) | 500ms | `HeroSection`, `LiveEventsSection`, `FeaturedEventsSection` | Above the fold, visible early but static fallback works |
| 2 (mid page) | 300ms after mount | `UpcomingEventsSection` | Mounts after TenantSettings loads (natural extra delay) |
| 3 (lower) | 800ms after mount | `TeamSection` | Further down page, mounts after TenantSettings |
| 4 (bottom) | 1500ms | `OurSponsorsSection` | Bottom of page, lowest priority |

### Timeline (First Visit, Cold Start)

```
0ms     — Server-side layout runs (auth checks on Amplify Lambda)
~500ms  — HTML arrives at browser, JS bundles start loading
~800ms  — React hydrates, static sections paint immediately
~830ms  — usePageReady fires → TenantSettingsProvider starts fetch
~1000ms — TenantSettings response → UpcomingEvents + Team mount
~1300ms — Hero/Live/Featured events data starts fetching
~1300ms — UpcomingEventsSection starts fetching (300ms after mount)
~1800ms — TeamSection starts fetching (800ms after mount)
~2330ms — OurSponsorsSection starts fetching
```

### Timeline (Repeat Visit, Cached)

```
0ms     — Page hydrates, static sections paint
~30ms   — Cache checks fire instantly → all cached data loads
          No network requests needed!
```

## Mandatory Pattern

### For Components That Fetch Data on the Home Page

```tsx
// ✅ DO: Always use deferred fetch pattern
import { useDeferredFetch } from '@/hooks/usePageReady';

const MySection: React.FC = () => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);

  // Choose delay based on section position (see tier table above)
  const shouldFetch = useDeferredFetch(800);

  useEffect(() => {
    async function loadData() {
      // 1. ALWAYS check sessionStorage cache first (instant, no delay)
      try {
        const cached = sessionStorage.getItem(CACHE_KEY);
        if (cached) {
          const { data, timestamp } = JSON.parse(cached);
          if (Date.now() - timestamp < CACHE_DURATION) {
            setData(data);
            setLoading(false);
            return; // Cache hit — no network request needed
          }
        }
      } catch { /* ignore */ }

      // 2. Gate network request behind deferred flag
      if (!shouldFetch) return;

      // 3. Now safe to make API call
      const response = await fetch('/api/proxy/...');
      // ...
    }

    loadData();
  }, [shouldFetch]); // Re-run when shouldFetch becomes true
};
```

```tsx
// ❌ DON'T: Fetch immediately on mount
useEffect(() => {
  fetch('/api/proxy/...');
}, []); // Fires during hydration — blocks static content!
```

### For Shared Hooks (useEventsData)

```tsx
// ✅ DO: Accept `enabled` parameter to defer
export const useEventsData = (enabled: boolean = true) => {
  useEffect(() => {
    if (!enabled) return; // Don't fetch until enabled
    fetchEventsData();
  }, [enabled]);
};

// In the component:
const shouldFetch = useDeferredFetch(500);
const { filteredEvents } = useFilteredEvents('hero', shouldFetch);
```

### For Context Providers (TenantSettingsProvider)

```tsx
// ✅ DO: Use usePageReady() directly (no extra delay — it's a gate)
const pageReady = usePageReady();

useEffect(() => {
  // Cache check first (instant)
  const cached = checkCache();
  if (cached) { setSettings(cached); return; }

  // Defer network request until after initial paint
  if (!pageReady && retryCount === 0) return;

  // Now fetch...
}, [pageReady, retryCount]);
```

## Key Rules

1. **NEVER fetch data in a `useEffect(() => {...}, [])` on the home page** — this fires during hydration and blocks paint.

2. **ALWAYS check sessionStorage cache before the deferral gate** — cached data should load instantly on repeat visits, regardless of the deferred flag.

3. **Use appropriate delay tiers** — above-the-fold sections get lower delays; below-the-fold sections get higher delays.

4. **The `enabled` parameter pattern for shared hooks** — hooks like `useEventsData` must accept an `enabled` flag so consuming components can control when data fetching starts.

5. **Cache duration is 5 minutes** — all homepage sections use `sessionStorage` with a 5-minute TTL. This means the second page view within 5 minutes is instant.

6. **Graceful degradation** — if an API call fails, the section renders a fallback UI or hides. It should never block other sections or throw unhandled errors.

7. **No server-side data fetching for public pages** — the home page is a `'use client'` component. Server-side fetching in `layout.tsx` should be limited to auth checks. All content data is loaded client-side with deferred patterns.

## AWS Amplify Considerations

- **Cold starts**: Amplify Lambda can take 1-3 seconds on cold start. Deferring API calls ensures the page shell renders before the Lambda is even warm.
- **`force-dynamic` in layout**: The root layout uses `export const dynamic = 'force-dynamic'` which means every request hits the Lambda. Minimizing server-side work is critical.
- **Connection limits**: Browsers limit to 6 concurrent connections per origin. Staggering prevents hitting this limit.
- **`cache: 'no-store'` on fetches**: All proxy API calls use `cache: 'no-store'` to avoid Next.js Data Cache issues on Amplify. The client-side sessionStorage cache handles caching instead.

## Files Implementing This Pattern

- `src/hooks/usePageReady.ts` — `usePageReady()` and `useDeferredFetch()` hooks
- `src/components/TenantSettingsProvider.tsx` — Uses `usePageReady()` to defer settings fetch
- `src/hooks/useEventsData.ts` — Accepts `enabled` param to defer event data fetch
- `src/hooks/useFilteredEvents.ts` — Passes `enabled` through to `useEventsData`
- `src/components/HeroSection.tsx` — `useDeferredFetch(500)` for hero events
- `src/components/LiveEventsSection.tsx` — `useDeferredFetch(500)` for live events
- `src/components/FeaturedEventsSection.tsx` — `useDeferredFetch(500)` for featured events
- `src/components/UpcomingEventsSection.tsx` — `useDeferredFetch(300)` for upcoming events
- `src/components/TeamSection.tsx` — `useDeferredFetch(800)` for team members
- `src/components/OurSponsorsSection.tsx` — `useDeferredFetch(1500)` for sponsors

## When Adding New Sections to the Home Page

1. Determine the section's vertical position on the page
2. Choose the appropriate delay tier from the table above
3. Use `useDeferredFetch(delayMs)` with the chosen delay
4. Always check sessionStorage cache before the deferred gate
5. Wrap the section in an `<ErrorBoundary>` with a fallback component
6. Test on a fresh browser session (no cache) to verify deferred behavior

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
