## gowater-hr-portal-monorepo

> > **Purpose:** Standard development rules and best practices

# GoWater Monorepo - Development Guidelines

> **Purpose:** Standard development rules and best practices
> **Last Updated:** 2026-03-18

> **Session Notes:** Water theme, P3 loading screen, page transitions, auth context, glass cards, TaskTimelineView compact layout

---

## Project Structure

```
gowater-monorepo/
├── apps/
│   ├── web/                 # Next.js 15 web application
│   └── mobile/              # Expo React Native application
├── packages/
│   └── types/               # Shared TypeScript types (@gowater/types)
├── infra/                   # Docker, Caddy, deployment configs (NOT YET CREATED — target architecture)
├── docs/                    # Documentation and references
│   ├── CODE_REFERENCE.md
│   ├── DATABASE_FIELDS_REFERENCE.md
│   ├── REUSABLE_REFERENCE.md
│   ├── TRANSITIONS_REFERENCE.md   # GSAP animation patterns catalog
│   ├── new-features.md            # P3 loading screen animation spec
│   └── persona-glass-cards.jsx    # Glass card style reference
└── CLAUDE.md                # This file
```

---

## Migration Status (Current → Target)

| Component            | Current (Production)               | Target (Hetzner VPS)                           | Status      |
| -------------------- | ---------------------------------- | ---------------------------------------------- | ----------- |
| Database             | Supabase (hosted PostgreSQL)       | Self-hosted PostgreSQL 16 in Docker            | Not started |
| Photo Storage        | Cloudinary                         | Hetzner Object Storage (S3-compatible)         | Not started |
| Watermarking         | Satori + Sharp → Cloudinary upload | Satori + Sharp → Hetzner Object Storage upload | Not started |
| Hosting              | Vercel (Next.js)                   | Hetzner VPS + Docker + Caddy                   | Not started |
| Infrastructure files | None (`infra/` dir empty)          | Dockerfile, docker-compose, Caddyfile, scripts | Not started |

**Why migrate from Cloudinary:** Cloudinary's transformation limits prevent custom watermark UI on photos (text overlay limitations). Satori+Sharp gives full JSX control over watermark design, and Hetzner Object Storage is cheaper for direct upload of pre-watermarked images.

**Important:** All `infra/` section content in this document describes the TARGET architecture. Do not assume these files or configs exist until this table is updated.

---

## Critical Rules

### 0. Use Opus 4.6

- **Always use Claude Opus 4.6** as the model for all AI-assisted development in this project
- Do not downgrade to Sonnet or Haiku for code generation tasks

### 1. No AI Attribution

- **Never mention AI, Claude, or any AI assistant** in commit messages, code comments, documentation, or anywhere in the codebase
- **No Co-Authored-By lines** referencing AI in commits
- Keep all contributions anonymous as standard developer work

### 2. Always Read Documentation First

Before making ANY changes, read the relevant docs:

- `docs/CODE_REFERENCE.md` - API endpoints, services, types
- `docs/DATABASE_FIELDS_REFERENCE.md` - Database schema, field names
- `docs/REUSABLE_REFERENCE.md` - Existing components, hooks, patterns

### 3. Never Hallucinate

- **Database fields:** Always use exact field names from `DATABASE_FIELDS_REFERENCE.md`
- **API endpoints:** Always verify endpoints exist in `CODE_REFERENCE.md`
- **Types:** Always import from `@gowater/types` or check existing type definitions
- **Components:** Check if component exists before creating new ones

### 4. Naming Conventions

| Context                 | Convention | Example                                      |
| ----------------------- | ---------- | -------------------------------------------- |
| Database fields         | snake_case | `user_id`, `check_in_time`, `break_duration` |
| TypeScript/JS variables | camelCase  | `userId`, `checkInTime`, `breakDuration`     |
| React components        | PascalCase | `TaskCard`, `AttendanceModal`                |
| CSS classes (web)       | kebab-case | `task-card`, `modal-header`                  |
| API routes              | kebab-case | `/api/attendance/edit-requests`              |
| File names (components) | PascalCase | `TaskCard.tsx`                               |
| File names (utilities)  | camelCase  | `formatDate.ts`                              |

### 5. Plan Before Executing

- State the full implementation plan before writing any code or modifying any file
- Get explicit approval before proceeding
- If scope changes mid-implementation, stop and re-plan

### 6. Production-Grade Clean Code

- No hacks, no shortcuts, no commented-out code, no dead code
- No bloating — do not add dependencies, abstractions, or utilities unless directly required
- No over-engineering — solve only what is asked, nothing more
- **Simple tasks require simple solutions** — if a feature is adding items to an existing sidebar list, don't create new pages/routes/layouts. Follow the existing pattern. If Leads/Events/Supplier are buttons that switch content inline, new categories must do the same — not navigate to separate pages
- Before creating new files, ask: can this be done by adding to an existing file? If yes, do that
- Before creating new routes, ask: does the existing page already handle similar content? If yes, add to it
- Don't create separate pages for content that belongs in the same view
- No over-engineering — solve only what is asked, nothing more
- No `any` types — use exact types from `@gowater/types` or define a precise local interface
- No `console.log` left in production code — only `logger.error()` / `logger.info()` via `src/lib/logger.ts`
- Every function does one thing

### 7. AI Temperature — Precision First

- **Development approach:** Claude must be precision-first and deterministic — follow existing patterns exactly, no creative liberties, no unsolicited refactoring or improvements
- **LLM calls in this codebase:** None currently (AI lives in n8n, not in app code). If ever added: structured output → `temperature: 0`, creative → `temperature: 0.7` max
- Never exceed `temperature: 0.7` in any LLM call

### 7. AI Temperature — Precision First

- **Development approach:** Claude must be precision-first and deterministic — follow existing patterns exactly, no creative liberties, no unsolicited refactoring or improvements
- **LLM calls in this codebase:** None currently (AI lives in n8n, not in app code). If ever added: structured output → `temperature: 0`, creative → `temperature: 0.7` max
- Never exceed `temperature: 0.7` in any LLM call

---

## Mobile App (Expo/React Native)

### File Structure

```
apps/mobile/
├── app/                     # Expo Router pages
│   ├── index.tsx           # Login screen (entry)
│   ├── _layout.tsx         # Root layout
│   └── (auth)/             # Protected routes
│       ├── _layout.tsx     # Tab layout
│       ├── dashboard.tsx
│       ├── attendance.tsx
│       └── tasks.tsx
├── src/
│   ├── contexts/           # React contexts
│   ├── services/           # API service wrappers
│   ├── components/         # Reusable components
│   ├── hooks/              # Custom hooks
│   └── utils/              # Utility functions
└── assets/                 # Images, fonts
```

### Component Patterns

#### Standard Screen Template

```tsx
import { useState, useCallback } from 'react';
import { View, Text, StyleSheet, ScrollView, RefreshControl } from 'react-native';

export default function ScreenName() {
    const [isLoading, setIsLoading] = useState(true);
    const [isRefreshing, setIsRefreshing] = useState(false);
    const [data, setData] = useState<DataType | null>(null);

    const fetchData = useCallback(async () => {
        try {
            const result = await service.getData();
            setData(result);
        } catch (error) {
            console.error('Failed to fetch data:', error);
        } finally {
            setIsLoading(false);
            setIsRefreshing(false);
        }
    }, []);

    const onRefresh = useCallback(() => {
        setIsRefreshing(true);
        fetchData();
    }, [fetchData]);

    if (isLoading) {
        return <LoadingView />;
    }

    return (
        <ScrollView
            style={styles.container}
            refreshControl={
                <RefreshControl
                    refreshing={isRefreshing}
                    onRefresh={onRefresh}
                    tintColor="#3b82f6"
                    colors={['#3b82f6']}
                />
            }
        >
            {/* Content */}
        </ScrollView>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#0f1824',
    },
});
```

#### Service Pattern

```tsx
// src/services/example.ts
import * as SecureStore from 'expo-secure-store';

const API_BASE_URL = process.env.EXPO_PUBLIC_API_URL || 'http://localhost:3000';

async function getAuthHeaders(): Promise<Record<string, string>> {
    const token = await SecureStore.getItemAsync('auth_token');
    return {
        'Content-Type': 'application/json',
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
    };
}

export const exampleService = {
    async getData(): Promise<DataType[]> {
        const headers = await getAuthHeaders();
        const response = await fetch(`${API_BASE_URL}/api/endpoint`, {
            method: 'GET',
            headers,
        });

        if (!response.ok) {
            throw new Error('Failed to fetch data');
        }

        const data = await response.json();
        return data.items || [];
    },
};
```

### Mobile UI Standards

#### Color Palette

```tsx
const colors = {
    // Backgrounds
    bgPrimary: '#0f1824', // Main background
    bgSecondary: '#1a2332', // Card background
    bgTertiary: '#374151', // Input background

    // Brand
    primary: '#3b82f6', // Blue - primary actions
    success: '#22c55e', // Green - success, check-in
    danger: '#ef4444', // Red - danger, check-out
    warning: '#f59e0b', // Orange - warnings, break

    // Text
    textPrimary: '#ffffff',
    textSecondary: '#9ca3af',
    textMuted: '#6b7280',

    // Status
    pending: '#fef3c7',
    inProgress: '#dbeafe',
    completed: '#dcfce7',
};
```

#### Spacing Standards

```tsx
const spacing = {
    xs: 4,
    sm: 8,
    md: 12,
    lg: 16,
    xl: 20,
    xxl: 24,
};
```

#### Typography

```tsx
const typography = {
    // Sizes
    xs: 10,
    sm: 12,
    base: 14,
    lg: 16,
    xl: 18,
    '2xl': 20,
    '3xl': 24,
    '4xl': 28,

    // Weights
    normal: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
};
```

### Modal Pattern (iOS Compatible)

```tsx
// IMPORTANT: iOS has issues with multiple modals
// Always close one modal before opening another

const openSecondModal = () => {
    setShowFirstModal(false);
    setTimeout(() => setShowSecondModal(true), 100);
};

const closeSecondModal = () => {
    setShowSecondModal(false);
    setTimeout(() => setShowFirstModal(true), 100);
};
```

---

## Web App (Next.js)

### File Structure

```
apps/web/
├── src/
│   ├── app/                # App Router pages
│   │   ├── api/           # API routes
│   │   ├── dashboard/     # Dashboard pages
│   │   └── layout.tsx     # Root layout
│   ├── components/        # React components
│   ├── contexts/          # React contexts
│   ├── hooks/             # Custom hooks
│   ├── lib/               # Services, utilities
│   └── types/             # TypeScript types
└── public/                # Static assets
```

### API Route Pattern

```tsx
// app/api/example/route.ts
import { NextRequest, NextResponse } from 'next/server';
import jwt from 'jsonwebtoken';
import { getDb } from '@/lib/db';

interface JWTPayload {
    userId: number;
    email: string;
    role: string;
}

function getTokenFromRequest(request: NextRequest): string | null {
    // Check cookies first (web), then Authorization header (mobile)
    let token = request.cookies.get('auth-token')?.value;

    if (!token) {
        const authHeader = request.headers.get('Authorization');
        if (authHeader?.startsWith('Bearer ')) {
            token = authHeader.substring(7);
        }
    }

    return token || null;
}

export async function GET(request: NextRequest) {
    try {
        const token = getTokenFromRequest(request);

        if (!token) {
            return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
        }

        const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
        const userId = decoded.userId;

        // Your logic here

        return NextResponse.json({ data });
    } catch (error) {
        return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
    }
}
```

### Auth Context (Critical)

- Auth is a shared React Context (`src/contexts/AuthContext.tsx`), NOT a standalone hook
- `useAuth()` re-exports from the context — all components share one auth state
- `AuthProvider` wraps the entire app in `src/app/layout.tsx`
- Dashboard layout handles auth gating — individual pages must NOT have their own `router.push('/auth/login')` redirects
- The context watches `pathname` changes and calls `verifyAuth()` when navigating to protected routes
- **Login page must NOT use `useAuth().login()`** — it bypasses the context and uses a raw `fetch('/api/auth/login')` directly. Setting `authState.user` via the context during login causes parent re-renders that unmount the P3 loading screen. The cookie is set server-side; the dashboard verifies it on mount via `verifyAuth()`
- **Never call `fetch('/api/auth/logout')` on login page mount** — it races with the login API and can clear the auth cookie after it's been set

### Dashboard Layout Architecture

- `PersistentShell` wraps all dashboard pages — sidebar + topbar never re-render
- `WaterBackground` canvas lives in `PersistentShell` — shared across all pages
- `PageTransition` component provides the transition root div
- Dashboard layout shows a diagonal curtain (`PixelDissolveLoader`) on first load after auth
- The curtain only plays once (tracked by `curtainPlayed` ref)

### PersistentShell Height Chain (CRITICAL — do not break)

The shell's content area uses a specific flex/overflow pattern that makes `h-full` work for child pages. **Do not change this pattern:**

```
Shell outer div:     flex-1 overflow-hidden ml-64 relative
  <main>:            h-full relative z-10
    <PageTransition>: height: 100% (inline style)
      {page}:         h-full flex flex-col
```

**Rules:**
- The shell content div MUST use `flex-1 overflow-hidden` — NOT `overflow-y-auto`, NOT fixed `height: calc()`
- `overflow-hidden` on the shell constrains children to available height — this is what makes `h-full` propagate
- `overflow-y-auto` on the shell would create a scroll context that breaks `h-full` for flex children
- Pages that need to fill the viewport (like Attendance) use `h-full flex flex-col` with `flex-1` on rows
- Pages that need scrolling (like Home) put `overflow-y-auto` on their own content div, not on the shell
- Never add `height: calc(100vh - Xpx)` to the shell — use `flex-1 overflow-hidden` instead
- Never put two `style={{}}` attributes on the same JSX element — the second silently overwrites the first

### Water Background Theme (Web Dashboard)

All dashboard pages use a shared animated water canvas background (`WaterBackground` component in `PersistentShell`). Page content must use transparent/glass styling.

| Element    | Value                         | Do NOT use        |
| ---------- | ----------------------------- | ----------------- |
| Card bg    | `rgba(255,255,255,0.05)`      | `bg-white`        |
| Title text | `#fff`                        | `text-gray-900`   |
| Body text  | `rgba(255,255,255,0.6)`       | `text-gray-600`   |
| Muted text | `rgba(255,255,255,0.4)`       | `text-gray-400`   |
| Borders    | `rgba(255,255,255,0.1)`       | `border-gray-200` |
| Accent     | `#7dd3fc` / `text-cyan-400`   | `text-blue-600`   |

Reference: `docs/persona-glass-cards.jsx`, `src/components/WaterBackground.tsx`

### GSAP Animation Rules

- GSAP is installed in `apps/web` — import with `import gsap from 'gsap'`
- `@gsap/react` is installed but `useGSAP` hook has issues with Next.js — use plain `useEffect` with cleanup instead
- Always use `gsap.fromTo()` instead of `gsap.from()` — React 18 Strict Mode double-mounts effects, leaving elements stuck at invisible "from" state
- Always clean up: `return () => { tl.kill(); }` in useEffect
- Use refs to prevent animation replay: track what has been animated with a ref, skip if already done
- Animation reference: `docs/TRANSITIONS_REFERENCE.md`

### Page Transitions

- Diagonal curtain transition plays when navigating between dashboard pages
- `PageTransitionContext` (`src/contexts/PageTransitionContext.tsx`) provides `navigateTo(href)`
- Sidebar uses `navigateTo()` instead of `<Link>` — this plays the curtain animation BEFORE navigating
- Transition panels must be large enough to cover the entire content area (18 panels, 300% height, -60% top)
- Panel colors must match the water background gradient

### P3 Loading Screen (Login to Dashboard)

- Persona 3 Reload-style calendar transition plays after login
- Component: `src/components/P3LoadingScreen.tsx`
- Reference HTML: `docs/persona3_v11_spacing_fix.html` — source of truth for positioning math
- Day positioning uses math: `STEP_X=95`, angle `-32deg`, `STEP_Y = STEP_X * Math.tan(-rad)`
- The loading screen runs ~10 seconds, then redirects to dashboard
- Visual reference: `docs/Persona-3-Reload02182024-103846-43673.jpg`
- Day labels align by **left edge** (`translateY(-50%)` only, not `translate(-50%,-50%)`) for a clean diagonal staircase
- The blue stripe is a **separate horizontal band** (NOT clip-path polygon) — tilted 12deg, height 90px, with feathered gradient edges and box-shadow glow
- Stripe animation fires when the current day is highlighted: `tl.to(stripe, ..., curIdx * 0.08 + 0.8)`
- Year/month labels and "Visit Tonsuya Site" watermark are positioned **outside** the stripe div (not inside — stripe height is too small)
- Use `useEffect` + `gsap.timeline` with cleanup, NOT `useGSAP` hook

---

## Shared Types (@gowater/types)

### Usage

```tsx
// Always import from @gowater/types for shared types
import { User, Task, AttendanceRecord } from '@gowater/types';

// For app-specific types, define locally
interface LocalComponentProps {
    // ...
}
```

### Adding New Types

1. Add to `packages/types/src/`
2. Export from `packages/types/src/index.ts`
3. Run `pnpm install` to update workspace links

---

## State Management

### Context Pattern

```tsx
// contexts/ExampleContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface ExampleContextType {
    data: DataType | null;
    setData: (data: DataType) => void;
}

const ExampleContext = createContext<ExampleContextType | undefined>(undefined);

export function ExampleProvider({ children }: { children: ReactNode }) {
    const [data, setData] = useState<DataType | null>(null);

    return <ExampleContext.Provider value={{ data, setData }}>{children}</ExampleContext.Provider>;
}

export function useExample() {
    const context = useContext(ExampleContext);
    if (!context) {
        throw new Error('useExample must be used within ExampleProvider');
    }
    return context;
}
```

---

## Error Handling

### Mobile

```tsx
try {
    const result = await service.doSomething();
    if (result.success) {
        // Handle success
    } else {
        Alert.alert('Error', result.error || 'Something went wrong');
    }
} catch (error) {
    console.error('Operation failed:', error);
    Alert.alert('Error', 'An unexpected error occurred');
}
```

### Web

```tsx
try {
    const result = await service.doSomething();
    if (!result.success) {
        toast.error(result.error || 'Something went wrong');
        return;
    }
    // Handle success
} catch (error) {
    console.error('Operation failed:', error);
    toast.error('An unexpected error occurred');
}
```

---

## Performance Best Practices

### 1. Memoization

```tsx
// Use useCallback for functions passed as props
const handlePress = useCallback(() => {
    // handler logic
}, [dependencies]);

// Use useMemo for expensive computations
const filteredData = useMemo(() => {
    return data.filter((item) => item.status === filter);
}, [data, filter]);
```

### 2. List Optimization

```tsx
// For long lists, use FlatList instead of ScrollView + map
<FlatList
    data={items}
    keyExtractor={(item) => item.id}
    renderItem={({ item }) => <ItemComponent item={item} />}
    initialNumToRender={10}
    maxToRenderPerBatch={10}
    windowSize={5}
/>
```

### 3. Image Optimization

```tsx
// Web: Use next/image
import Image from 'next/image';
<Image src="/image.png" alt="..." width={100} height={100} />;

// Mobile: Use expo-image for better performance
import { Image } from 'expo-image';
<Image source={{ uri: imageUrl }} style={styles.image} />;
```

---

## Security Guidelines

### 1. Token Storage

- **Mobile:** Use `expo-secure-store` (encrypted storage)
- **Web:** Use httpOnly cookies (set by server)

### 2. API Security

- Always validate JWT tokens on API routes
- Always check user ownership before data access
- Never expose sensitive data in error messages

### 3. Input Validation

- Validate all user inputs before API calls
- Sanitize data before database operations
- Use parameterized queries (never interpolate user input into SQL)

### 4. Infrastructure Security (Hetzner VPS)

- **SSH**: Key-only authentication, disable password auth, disable root login, non-standard port
- **Firewall (UFW)**: Default deny incoming, allow only 80, 443, and SSH port
- **fail2ban**: Enabled for SSH, Caddy, and Docker logs
- **Docker network isolation**: Each client's containers on their own Docker network, only Caddy connects to all
- **Secrets**: Never in code, never in Dockerfile. Use `.env` files with `600` permissions, owned by deploy user. Listed in `.gitignore`
- **Container security**: Non-root users, read-only filesystem where possible (`read_only: true`), drop all capabilities and add only needed ones
- **Automated backups**: Daily PostgreSQL `pg_dump` to Hetzner Object Storage, 30-day retention
- **Updates**: Unattended-upgrades for OS security patches, manual Docker image updates
- **Logging**: Docker logs with `json-file` driver, max-size 10MB, max-file 3

---

## Git Workflow

### Commit Messages

```
feat(mobile): add pull-to-refresh to dashboard
fix(web): resolve timezone issue in attendance
refactor(types): reorganize shared type exports
docs: update API reference documentation
```

### Branch Naming

```
feature/mobile-checkout-flow
fix/attendance-timezone
refactor/shared-components
```

---

## Testing Checklist

Before marking a feature complete:

- [ ] Works on iOS (Expo Go)
- [ ] Works on Android (Expo Go)
- [ ] Works on Web (if applicable)
- [ ] Pull-to-refresh works
- [ ] Error states handled
- [ ] Loading states shown
- [ ] No console errors/warnings
- [ ] API errors handled gracefully

---

## Infrastructure & Deployment (Hetzner VPS) — TARGET ARCHITECTURE

> **Status:** Not yet implemented. See Migration Status table above. Current production runs on Vercel + Supabase + Cloudinary.

### VPS Architecture

- **Provider:** Hetzner Cloud, CX32 Singapore (4 vCPU, 8GB RAM, ~$7.50/mo)
- **OS:** Ubuntu 24.04 LTS
- Each client project gets its own `docker-compose.yml` with isolated containers
- Shared reverse proxy (Caddy) handles SSL and routing for all client domains
- Shared services: Caddy only. Each client gets own PostgreSQL + Next.js containers

### Docker Rules

- Every service runs in a container — no bare-metal processes
- One `Dockerfile.web` per app, multi-stage build (builder → runner)
- Non-root user inside containers (`USER node` / `USER nextjs`)
- Named volumes for persistent data (PostgreSQL, uploads)
- Resource limits on every container (`mem_limit`, `cpus`)
- `restart: unless-stopped` on all production containers
- No `latest` tags — pin image versions (e.g., `node:20-alpine`, `postgres:16-alpine`)
- `.dockerignore` must exclude `node_modules`, `.git`, `.env*`, `*.md`
- Health checks on every container

### Caddy Reverse Proxy Rules

- One Caddyfile for all client domains
- Automatic HTTPS via Let's Encrypt
- Security headers (HSTS, X-Frame-Options, etc.) applied at Caddy level, not Next.js
- Rate limiting on API routes
- Pattern:

```
portal.gowatervendo.com {
    reverse_proxy gowater-web:3000
}
```

### Database Rules (Self-hosted PostgreSQL)

- PostgreSQL 16 Alpine in Docker
- Each client gets their own PostgreSQL container and named volume
- Backups: daily `pg_dump` compressed, stored on Hetzner Object Storage
- Connection only from app container on same Docker network — never exposed to host
- Strong password, not default `postgres` user

### Photo Storage (Hetzner Object Storage)

- S3-compatible API
- Photos uploaded via `@aws-sdk/client-s3` (works with any S3-compatible storage)
- Already-watermarked images uploaded (no server-side transforms needed on storage)
- Public read bucket for photo URLs, private write via IAM credentials
- CDN (Cloudflare or Bunny) in front for caching

---

## Multi-Client Architecture

### VPS Directory Structure

```
/srv/
├── shared/
│   └── docker-compose.yml    # Caddy reverse proxy
│       └── caddy/Caddyfile
├── clients/
│   ├── gowater/
│   │   ├── docker-compose.yml
│   │   ├── .env
│   │   └── data/             # Mounted volumes
│   ├── client-2/
│   │   ├── docker-compose.yml
│   │   ├── .env
│   │   └── data/
│   └── ...
└── backups/
    └── scripts/
        └── backup.sh
```

### Client Onboarding Rules

1. Create client folder under `/srv/clients/<client-name>/`
2. Copy template `docker-compose.yml` and customize domain, ports, env
3. Add domain block to shared Caddyfile
4. Run `docker compose up -d` in client folder
5. Run database migrations
6. Caddy auto-provisions SSL certificate

### Client Isolation Rules

- Each client: own Docker network, own PostgreSQL, own app container, own volumes
- No shared databases — ever
- No cross-client container communication
- Caddy is the only container that bridges networks

---

## Photo Watermarking (Satori + Sharp)

Watermarks are generated server-side using **Satori** (JSX → SVG) and **Sharp** (composite SVG onto photo). Photos are uploaded already-watermarked to Hetzner Object Storage.

### Architecture

- Watermark templates live in `apps/web/src/lib/watermark/`
- One JSX component per watermark type: checkin, checkout, break
- Satori renders JSX to SVG with full CSS flexbox layout — no Cloudinary transformation limits
- Sharp composites the SVG watermark onto the original photo buffer
- The watermarked image is then uploaded to S3-compatible storage (Hetzner Object Storage)

### Hard Rules

1. **Watermark before upload** — never rely on storage-side transforms. The photo uploaded to object storage must already contain the watermark.
2. **Satori JSX must use inline styles only** — Satori does not support external CSS, class names, or `styled-components`. All styles must be inline `style={{ }}` objects.
3. **Satori supports a subset of CSS** — flexbox layout, basic text styling, colors, borders, padding, margin. No CSS Grid, no `position: absolute` relative to viewport, no animations.
4. **Sharp composite order matters** — composite the watermark SVG onto the photo, not the other way around. Use `sharp(photoBuffer).composite([{ input: svgBuffer, gravity: 'south' }])`.
5. **Philippines timezone (UTC+8)**: Always use manual offset `new Date(now.getTime() + (8 * 60 * 60 * 1000))` with `getUTCHours()`/`getUTCMinutes()`. Do NOT use `toLocaleTimeString` — it's unreliable in Docker/server environments.
6. **Font loading** — Satori requires font data as `ArrayBuffer`. Load fonts once at module level, not per-request. Store font files in `apps/web/src/lib/watermark/fonts/`.
7. **SVG dimensions must match photo width** — render the watermark SVG at the same width as the source photo to avoid scaling artifacts.

---

## Common Pitfalls to Avoid

1. **Don't create duplicate components** - Check `REUSABLE_REFERENCE.md` first
2. **Don't hardcode API URLs** - Use environment variables
3. **Don't skip loading states** - Always show loading indicators
4. **Don't ignore errors** - Always handle and display errors
5. **Don't use inline styles excessively** - Use StyleSheet.create()
6. **Don't forget pull-to-refresh** - Add to all scrollable screens
7. **Don't nest modals on iOS** - Close one before opening another
8. **Don't forget type safety** - Always define proper TypeScript types
9. **Don't expose Docker ports to host** - Use Caddy reverse proxy instead
10. **Don't store secrets in Dockerfiles or code** - Use `.env` files only
11. **Don't use `latest` Docker image tags** - Pin versions (e.g., `node:20-alpine`)
12. **Don't run containers as root** - Use `USER node` in Dockerfile
13. **Don't share databases between clients** - Full isolation always
14. **Don't use `ON CONFLICT` in SQL migrations** - Supabase tables may lack UNIQUE constraints even when defined in `CREATE TABLE IF NOT EXISTS` (the statement is skipped if table exists, so constraints are never applied). Always use `DO $$ IF NOT EXISTS (SELECT 1 FROM table WHERE col = 'value') THEN INSERT ... END IF; END $$;` instead
15. **Don't assume `CREATE TABLE IF NOT EXISTS` applies constraints** - If the table already exists, PostgreSQL skips the entire statement including all constraints, indexes, and defaults defined in it. Add constraints separately with `ALTER TABLE ... ADD CONSTRAINT IF NOT EXISTS` or use DO blocks
16. **Single migration file** - All DB migrations go in `apps/web/migrations/run_all_migrations.sql`. Do not create separate migration files. All statements must be idempotent (safe to run multiple times)
17. **Don't use `useGSAP` from `@gsap/react`** - It silently fails in some Next.js setups. Use plain `useEffect` + direct timeline with cleanup
18. **Don't use `gsap.from()` in React** - Always use `gsap.fromTo()` to avoid React 18 Strict Mode double-mount issues
19. **Don't add auth redirects in individual dashboard pages** - The dashboard layout handles auth gating via `AuthContext`
20. **Don't use `bg-white` or solid backgrounds on dashboard pages** - The water background must show through; use glass/transparent styles
21. **Don't use `<Link>` for dashboard sidebar navigation** - Use `navigateTo()` from `PageTransitionContext` so the diagonal curtain transition plays first
22. **Page transition panels must be oversized** - Use 300% height with -60% top offset to fully cover screen at -32deg rotation
23. **Don't use `overflow-y-auto` on PersistentShell content div** - It breaks `h-full` propagation for child pages. Use `flex-1 overflow-hidden` on the shell, let each page manage its own scroll
24. **Don't put two `style={{}}` on the same JSX element** - The second silently overwrites the first. Merge all styles into one object
25. **Don't use `height: calc(100vh - Xpx)` on the shell** - Use `flex-1 overflow-hidden` instead. Fixed heights break the flex chain
26. **Don't use `useAuth().login()` on the login page** - It sets `authState.user` which triggers parent re-renders and unmounts the P3 loading screen. Use raw `fetch('/api/auth/login')` instead. The cookie is set server-side; dashboard verifies on mount
27. **Don't call `fetch('/api/auth/logout')` on login page mount** - It races with the login API and clears the cookie
28. **Don't create separate pages/routes for content that belongs in an existing view** - If Leads/Events/Supplier are sidebar buttons on one page, new categories (Cold Leads sub-types) should be sidebar buttons on the same page, not separate routes
29. **Don't mix `leads` and `cold_leads` table columns** - The `leads` table does NOT have a `cold_category` column. The `cold_leads` table DOES. Regular Leads/Events/Supplier use `LeadService` → `/api/leads` → `leads` table. Cold Leads (Restaurants, LGU, Hotel, Microfinance, Foundation) use `ColdLeadService` → `/api/cold-leads` → `cold_leads` table. Never insert `cold_category` into the `leads` table
    - **Zod strips unknown fields** — if you add a new field to `LeadFormData`, you MUST also add it to `createLeadSchema` in `src/lib/validation/schemas.ts`. Zod's discriminated union silently removes fields not in the schema. `cold_category` is in the `baseLeadSchema`
30. **Don't use `toISOString().split('T')[0]` to get a date string for API queries** - `toISOString()` converts to UTC, which shifts the date back one day in PH timezone (UTC+8). Midnight local PH = previous day 16:00 UTC. Always use local date formatting: `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`. This applies to any date used in API query parameters, date range filters, or date comparisons
31. **Don't use `new Date("YYYY-MM-DD")` — Safari returns Invalid Date** - Safari rejects date-only strings and PostgreSQL timestamptz format (`"2026-03-21 01:00:00+00"`). Always use `safeParse()` from `src/lib/timezone.ts` which handles both formats. For date-only strings append `T00:00:00`, for PG timestamptz replace the space with `T`. Chrome is lenient but Safari is strict — always test date parsing in Safari
32. **Philippines timezone (UTC+8)** - Server-side (Vercel runs UTC): use `getPhilippineDateString()` from `src/lib/timezone.ts`. Client-side: use `toLocalDateStr()` helper. Never rely on `new Date().toISOString()` for dates — it produces UTC dates which are wrong for PH after 4 PM UTC (midnight PH)

---

## Quick Reference Commands

```bash
# Local development
pnpm install
pnpm run dev:web
cd apps/mobile && npx expo start --clear

# Type check
cd apps/web && npx tsc --noEmit
cd apps/mobile && npx tsc --noEmit

# Docker (production)
docker compose build
docker compose up -d
docker compose logs -f web
docker compose exec postgres pg_dump -U gowater gowater > backup.sql

# Deploy update
docker compose pull && docker compose up -d

# Per-client management
cd /srv/clients/gowater && docker compose restart
cd /srv/clients/gowater && docker compose logs -f
```

---

## Environment Variables

### Mobile (.env)

```
EXPO_PUBLIC_API_URL=http://YOUR_IP:3000
```

### Web (.env)

```
# Database (self-hosted PostgreSQL)
DATABASE_URL=postgresql://gowater:password@postgres:5432/gowater

# Auth
JWT_SECRET=...

# Hetzner Object Storage (S3-compatible)
S3_ENDPOINT=https://fsn1.your-objectstorage.com
S3_BUCKET=gowater-photos
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
S3_REGION=fsn1
```

---

_This document should be updated as new patterns and conventions are established._

---
> Source: [tekkenfreya/gowater-hr-portal-monorepo](https://github.com/tekkenfreya/gowater-hr-portal-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
