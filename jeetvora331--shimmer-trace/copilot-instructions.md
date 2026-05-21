## shimmer-trace

> Use when: component renders fine with real data, just need skeleton while loading.

# shimmer-trace — Agent & LLM Reference

> This file is written for AI agents, LLMs, and prompt engineers.
> It gives you everything needed to correctly generate, debug, and reason about shimmer-trace usage.
> Human-readable README is at `README.md`.

---

## What This Library Does (One Paragraph)

`shimmer-trace` is a React skeleton loading library. It renders your real component invisibly, walks the live DOM to find every visible leaf element (headings, paragraphs, images, inputs, buttons, etc.), measures each one's exact position and size via `getBoundingClientRect()`, then paints an absolutely-positioned shimmer overlay on top of those measured rects. The skeleton is therefore pixel-perfect and auto-updates via `ResizeObserver`. You never write a manual skeleton. You wrap your component and pass `loading={true}`.

---

## Install

```bash
npm install shimmer-trace
```

**Peer deps required:** `react >= 18.0.0`, `react-dom >= 18.0.0`

---

## Public API — Full Export List

```ts
// Components
import { Shimmer } from 'shimmer-trace'; // main component
import { ShimmerSuspense } from 'shimmer-trace'; // Suspense boundary with auto-skeleton
import { createShimmer } from 'shimmer-trace'; // factory — bakes config into component

// Hooks
import { useIsShimmering } from 'shimmer-trace'; // true when inside ShimmerSuspense fallback
import { useShimmerContext } from 'shimmer-trace'; // raw context value — advanced use only

// Raw context (rarely needed — only if you build a custom Master/Reporter outside <Shimmer>)
import { ShimmerContext } from 'shimmer-trace';

// Types
import type {
  ShimmerProps,
  ShimmerConfig,
  ShimmerRect,
  AnimationType,
  ShimmerSuspenseProps,
} from 'shimmer-trace';
```

---

## Types Reference

### `AnimationType`

```ts
type AnimationType = 'wave' | 'pulse' | 'shine' | 'glow' | 'gradient';
```

| Value | Behaviour |
|---|---|
| `wave` | Horizontal gradient sweep left→right across full container width. Default. |
| `pulse` | Opacity oscillates 0.4→1→0.4. No movement. |
| `shine` | Diagonal sweep (115°) with skew — more premium feel than wave. |
| `glow` | Brightness oscillates 1→1.35→1 via CSS filter. |
| `gradient` | Block background itself animates as sliding gradient (no child layer). |

### `ShimmerConfig`

All fields optional. Used by `Shimmer` props, `createShimmer`, and `ShimmerSuspense`.

```ts
interface ShimmerConfig {
  animation?:          AnimationType;  // default: 'wave'
  baseColor?:          string;         // default: '#e0e0e0'  (CSS color, any format)
  highlightColor?:     string;         // default: '#f5f5f5'  (CSS color, any format)
  speed?:              number;         // default: 1.5        (seconds, float)
  borderRadius?:       string;         // default: ''         (CSS value e.g. '8px'. Empty = auto-detect)
  preserveBackground?: boolean;        // default: true
}
```

**`preserveBackground` explained:**
- `true` (default): Master container stays visible. CSS rules hide text (`color:transparent`) and media (`opacity:0`) on leaf tags while keeping div backgrounds, borders, padding visible.
- `false`: Legacy mode. Master container gets `visibility:hidden`. Everything hidden. Overlay punches through with `visibility:visible`.

### `ShimmerProps`

Extends `ShimmerConfig`. All `ShimmerConfig` fields inherit plus:

```ts
interface ShimmerProps extends ShimmerConfig {
  loading?:           boolean;                    // default: false
  children:           ReactNode;                  // required
  dummyLength?:       number;                     // clone count for list mode
  dummyData?:         Record<string, any>;        // props merged into children while loading
  as?:                React.ComponentType<any>;   // component type for skeleton shape
  stopPropagation?:   boolean;                    // force Master even when nested
  className?:         string;                     // applied to Master container div
  style?:             React.CSSProperties;        // merged into Master container div
}
```

### `ShimmerSuspenseProps`

```ts
interface ShimmerSuspenseProps extends ShimmerConfig {
  children:   ReactNode;
  template?:  ReactNode;  // explicit skeleton shape; if omitted uses useIsShimmering pattern
}
```

### `ShimmerRect` (internal, rarely needed by consumers)

```ts
interface ShimmerRect {
  x:            number;  // left offset from Master container
  y:            number;  // top offset from Master container
  width:        number;
  height:       number;
  borderRadius: string;
}
```

---

## Architecture — How It Works Internally

```
<Shimmer loading={true}>           ← MasterShimmer
  children render hidden           ← visibility:hidden OR color:transparent
  useTrace() walks DOM             ← collectTraceableElements → getBoundingClientRect
  ResizeObserver re-traces         ← on container resize
  <ShimmerOverlay rects={...} />   ← absolutely positioned, z-index:1
    one <div> per traced rect      ← base color + optional SweepLayer child
    SweepLayer spans container     ← gradient child, full container width, synced wave

  <Shimmer> nested inside          ← ReporterShimmer (auto-detected via context)
    useTrace() walks own DOM       ← measures relative to Master container
    register(id, rects) to Master  ← rects bubble up to single overlay
    unregister on unmount          ← cleanup
```

**Traceable elements** (auto-detected by tag name):
`H1 H2 H3 H4 H5 H6 P SPAN A LI LABEL TD TH BLOCKQUOTE CODE PRE IMG VIDEO SVG CANVAS PICTURE INPUT TEXTAREA SELECT BUTTON HR`

Plus any element with `data-shimmer` attribute, or leaf elements (no children) with non-zero dimensions.

**Ignored elements:**
- Any element with `data-shimmer-ignore` attribute
- Any element with `data-shimmer-reporter` attribute (Reporter wrappers)

**Fallback dimensions** (used when element has zero size):
`INPUT→200×36 BUTTON→120×36 TEXTAREA→300×80 SELECT→200×36 IMG→100×100 H1→300×36 H2→260×30 H3→220×26 H4→200×22 H5→180×20 H6→160×18 P→250×16 SPAN→100×16`

---

## Pattern 1 — Basic Wrap (Most Common)

Use when: component renders fine with real data, just need skeleton while loading.

```tsx
import { Shimmer } from 'shimmer-trace';

function ProfilePage() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser().then(u => { setUser(u); setLoading(false); });
  }, []);

  return (
    <Shimmer loading={loading}>
      <UserCard user={user} />
    </Shimmer>
  );
}
```

**What happens:** while `loading=true`, `UserCard` renders with `user=null`. shimmer-trace traces whatever DOM `UserCard` produces and paints shimmer over it.

**Problem:** if `UserCard` renders nothing when `user=null` (early return, conditional rendering), skeleton is empty.

**Solution:** use `dummyData`.

---

## Pattern 2 — `dummyData` (Recommended for Null-Safe Components)

Use when: component conditionally renders / early-returns when data is null.

```tsx
const userTemplate = {
  name:   'Loading name',
  role:   'Loading role',
  bio:    'Loading bio text here',
  avatar: '',
};

<Shimmer loading={loading} dummyData={{ user: userTemplate }}>
  <UserCard user={user} />
</Shimmer>
```

**What happens:** while `loading=true`, `UserCard` is cloned with `{ user: userTemplate }` merged as props. Component renders with dummy data → full DOM → shimmer traces it.

**Rule:** keys in `dummyData` must match prop names of the direct child component.

---

## Pattern 3 — `as` Prop (Clean List Pattern)

Use when: rendering a list where real data starts empty `[]` and you want N skeleton cards.

```tsx
const movieTemplate = {
  movie: {
    id:           0,
    title:        'Loading title',
    poster_path:  '',
    release_date: '0000-00-00',
  }
};

<Shimmer
  loading={loading}
  as={MovieCard}
  dummyData={movieTemplate}
  dummyLength={10}
  className="movies-grid"
>
  {movies.map(m => <MovieCard movie={m} key={m.id} />)}
</Shimmer>
```

**What happens:** while `loading=true`, children ignored. Renders `10` instances of `<MovieCard {...movieTemplate} />`. shimmer traces them. When `loading=false`, real `movies.map(...)` renders.

**`as` vs `dummyData` distinction:**
- `as={MovieCard}` + `dummyData` + `dummyLength` → renders N instances of the component, ignores children
- `dummyData` only (no `as`) → clones existing children with dummy props merged in

---

## Pattern 4 — `dummyLength` Without `as` (Inline List)

Use when: children already render a list, data starts empty, no separate component ref needed.

```tsx
const items = loading ? Array(5).fill({ name: 'Loading', price: '$0.00' }) : realItems;

<Shimmer loading={loading}>
  {items.map((item, i) => (
    <ItemCard item={item} key={i} />
  ))}
</Shimmer>
```

Or with `dummyLength` on first child clone:

```tsx
<Shimmer loading={loading} dummyLength={5} dummyData={{ item: { name: 'Loading', price: '$0.00' } }}>
  <ItemCard item={realItem} />
</Shimmer>
```

---

## Pattern 5 — Nested Shimmer (Single Sync Wave)

Use when: multiple sub-sections exist but you want one synchronized shimmer across all.

```tsx
// Master wraps everything — one overlay, one wave
<Shimmer loading={loading} style={{ display: 'flex', gap: '1rem' }}>
  <StatCard value="4,821" label="Users" />
  <StatCard value="98.4%" label="Uptime" />
  <StatCard value="142ms" label="Latency" />
</Shimmer>

// Reporter auto-detected when Shimmer is nested inside another Shimmer
<Shimmer loading={loading}>
  <Header />
  <Shimmer>         {/* ← auto-becomes Reporter, bubbles rects to Master above */}
    <Sidebar />
  </Shimmer>
  <Content />
</Shimmer>
```

**Key rule:** Any nested `<Shimmer>` inside an outer `<Shimmer>` auto-becomes a Reporter, regardless of whether it has its own `loading` prop. The Reporter measures its own subtree, sends rects to the Master, and ignores its own `loading` prop — Master's `loading` controls everything. One overlay covers all. To break out and create an independent Master, use `stopPropagation={true}` (see Pattern 8).

---

## Pattern 6 — `createShimmer` Factory

Use when: same config needed in many places. Avoids prop repetition.

```tsx
import { createShimmer } from 'shimmer-trace';

// Define once — usually in a shared file
const AppShimmer = createShimmer({
  baseColor:       '#1e1e3a',
  highlightColor:  '#2d2d52',
  animation:       'shine',
  speed:           1.2,
});

// Use like a normal component anywhere
<AppShimmer loading={loading}>
  <UserCard user={user} />
</AppShimmer>

// Props still overridable per-instance
<AppShimmer loading={loading} animation="glow" dummyLength={5} as={MovieCard} dummyData={template}>
  {movies.map(m => <MovieCard movie={m} key={m.id} />)}
</AppShimmer>
```

---

## Pattern 7 — `ShimmerSuspense` (React Suspense Integration)

Use when: component uses `use()`, `useSuspenseQuery`, or similar — throws a Promise while loading.

### Option A — `template` prop (component has zero shimmer awareness)

Reuse the same component as its own template — pass template props so it renders DOM without suspending. No duplicate skeleton component, no `&nbsp;` width padding.

```tsx
import { ShimmerSuspense } from 'shimmer-trace';

// Template data: same shape the real component expects, no fetch
const userTemplate = {
  name:   'xxxxxxxxxxxxxx',
  role:   'xxxxxxxxxx',
  bio:    'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
  avatar: '',
};

<ShimmerSuspense template={<UserCard user={userTemplate} />} animation="wave">
  <UserCard />   {/* throws Promise while fetching — shimmer shows automatically */}
</ShimmerSuspense>
```

**Why a `template` prop at all (not just `dummyData` like `<Shimmer>`):** the real `<UserCard />` throws a Promise during render — it never produces DOM until data resolves. `cloneElement` + props merge doesn't help because the cloned element still suspends. `template` renders a separate, non-suspending instance (same component, template data) so Shimmer gets real DOM to trace.

**Rule:** template should render synchronously. If you pass `<UserCard resource={realResource} />` as template, it'll suspend inside the fallback and skeleton goes empty. Either pass template data props (above) or use Option B.


### Option B — `useIsShimmering` hook (component skips fetch in shimmer mode)

```tsx
import { ShimmerSuspense, useIsShimmering } from 'shimmer-trace';

function UserCard() {
  const isShimmering = useIsShimmering();

  // Skip the suspending call when rendering as skeleton shape
  const user = isShimmering ? null : use(userPromise);

  return (
    <div className="profile-card">
      <img className="avatar" src={user?.avatar ?? ''} alt="" />
      <div className="info">
        <h3>{user?.name ?? ' '}</h3>
        <span>{user?.role ?? ' '}</span>
        <p>{user?.bio ?? '            '}</p>
      </div>
    </div>
  );
}

<ShimmerSuspense animation="shine">
  <UserCard />
</ShimmerSuspense>
```

**Which option to use:**
- Option A → component is a library component or you can't modify it
- Option B → you own the component and want it to self-describe its skeleton shape

---

## Pattern 8 — `stopPropagation` (Force Independent Master)

Use when: nested Shimmer should NOT bubble rects to parent — it manages its own overlay.

```tsx
<Shimmer loading={outerLoading}>
  <Header />
  <Shimmer loading={innerLoading} stopPropagation={true}>
    {/* This is a fully independent Master, not a Reporter */}
    <Widget />
  </Shimmer>
</Shimmer>
```

---

## Pattern 9 — `data-shimmer` / `data-shimmer-ignore` Escape Hatches

```tsx
// Force trace on a custom element not in the tag whitelist
<div data-shimmer>
  <CustomCanvas />
</div>

// Exclude an element from tracing (e.g. decorative icon)
<span data-shimmer-ignore>
  <DecorativeIcon />
</span>
```

---

## Config Inheritance Chain

When props are omitted, they resolve in this order:

```
prop value → parent Shimmer context config → DEFAULTS
```

`DEFAULTS`:
```ts
{
  animation:          'wave',
  baseColor:          '#e0e0e0',
  highlightColor:     '#f5f5f5',
  speed:              1.5,
  borderRadius:       '',       // empty = auto-detect per element
  preserveBackground: true,
}
```

---

## Decision Tree — Which Pattern to Use

```
Need skeleton loader?
│
├─ Component uses Suspense (throws Promise)?
│   └─ YES → ShimmerSuspense
│       ├─ Can modify component? → Option B (useIsShimmering)
│       └─ Cannot modify? → Option A (template prop)
│
└─ NO → Shimmer with loading prop
    │
    ├─ List of N items, data starts empty []?
    │   └─ Use as={MyCard} + dummyData + dummyLength
    │
    ├─ Single component, renders null when data=null?
    │   └─ Use dummyData with template values
    │
    ├─ Single component, renders partial shape with null data?
    │   └─ Plain wrap, no dummyData needed
    │
    ├─ Same config used across many places?
    │   └─ createShimmer factory, use factory component
    │
    └─ Multiple nested Shimmers, want one sync wave?
        └─ Wrap all in single Master <Shimmer>, inner become Reporters automatically
```

---

## Common Mistakes Agents Must Avoid

### ❌ Wrong — `dummyData` key doesn't match prop name

```tsx
// Component receives prop named `user`
<UserCard user={data} />

// WRONG — key is `userData`, not `user`
<Shimmer loading={loading} dummyData={{ userData: template }}>
  <UserCard user={data} />
</Shimmer>

// CORRECT — key matches prop name exactly
<Shimmer loading={loading} dummyData={{ user: template }}>
  <UserCard user={data} />
</Shimmer>
```

### ❌ Wrong — using `as` without `dummyData` when component needs props

```tsx
// WRONG — MovieCard will crash because movie=undefined
<Shimmer loading={loading} as={MovieCard} dummyLength={5}>
  ...
</Shimmer>

// CORRECT
<Shimmer loading={loading} as={MovieCard} dummyData={{ movie: movieTemplate }} dummyLength={5}>
  ...
</Shimmer>
```

### ❌ Wrong — forgetting that `as` prop ignores `children` during loading

```tsx
// WRONG — dev expects children to also show during loading
<Shimmer loading={loading} as={SkeletonShape}>
  <RealContent />   {/* This is ignored while loading=true when `as` is set */}
</Shimmer>

// CORRECT — `as` is for the skeleton shape only; real children render when loading=false
// That is the intended behavior — not a bug.
```

### ❌ Wrong — `ShimmerSuspense` Option B without `useIsShimmering` guard

```tsx
// WRONG — component always calls use(promise), throws inside fallback, empty skeleton
function UserCard() {
  const user = use(userPromise);  // throws in fallback → skeleton has no shape
  return <div><h3>{user.name}</h3></div>;
}

// CORRECT
function UserCard() {
  const isShimmering = useIsShimmering();
  const user = isShimmering ? null : use(userPromise);
  return <div><h3>{user?.name ?? ' '}</h3></div>;
}
```

### ❌ Wrong — expecting `display:flex` on children, not on Shimmer container

```tsx
// WRONG — Shimmer container is position:relative, flex on children has no effect on layout
<div style={{ display: 'flex' }}>
  <Shimmer loading={loading}>
    <Card /><Card /><Card />
  </Shimmer>
</div>

// CORRECT — put flex on Shimmer itself via style prop
<Shimmer loading={loading} style={{ display: 'flex', gap: '1rem' }}>
  <Card /><Card /><Card />
</Shimmer>
```

### ❌ Wrong — passing `loading` to inner nested Shimmer expecting independent behavior

```tsx
// This creates a Reporter (ignores its own loading prop), not an independent Master
<Shimmer loading={outerLoading}>
  <Shimmer loading={innerLoading}>   {/* loading ignored — behaves as Reporter */}
    <Widget />
  </Shimmer>
</Shimmer>

// To make it independent, add stopPropagation
<Shimmer loading={outerLoading}>
  <Shimmer loading={innerLoading} stopPropagation={true}>
    <Widget />
  </Shimmer>
</Shimmer>
```

---

## CSS Side Effects

Library injects one `<style>` tag with id `shimmer-trace-styles` into `document.head` on first render. Contains only `@keyframes` definitions and `preserveBackground` CSS attribute selectors. Safe to SSR — guard is `typeof document === 'undefined'`.

---

## TypeScript Usage

```ts
import type { AnimationType, ShimmerConfig, ShimmerProps } from 'shimmer-trace';

// Type a config object
const config: ShimmerConfig = {
  animation: 'shine',
  baseColor: '#1a1a2e',
  speed: 1.2,
};

// Type guard for animation values
const anim: AnimationType = 'wave'; // 'wave'|'pulse'|'shine'|'glow'|'gradient'
```

---

## Full Working Example — Movie App

```tsx
import { Shimmer, createShimmer } from 'shimmer-trace';

// Themed factory
const DarkShimmer = createShimmer({
  baseColor:      '#1e1e3a',
  highlightColor: '#2d2d52',
  animation:      'wave',
});

// Template data matching MovieCard's prop shape
const movieTemplate = {
  movie: {
    id:           0,
    title:        'Loading title',
    poster_path:  '',
    release_date: '0000-00-00',
    vote_average: 0,
  }
};

function MoviePage() {
  const [movies, setMovies] = useState<Movie[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    getPopularMovies()
      .then(setMovies)
      .finally(() => setLoading(false));
  }, []);

  return (
    <DarkShimmer
      loading={loading}
      as={MovieCard}
      dummyData={movieTemplate}
      dummyLength={12}
      className="movies-grid"
    >
      {movies.map(m => <MovieCard movie={m} key={m.id} />)}
    </DarkShimmer>
  );
}
```

---

## Minimal Reproducible Snippet (for debugging)

```tsx
import { Shimmer } from 'shimmer-trace';

export default function Debug() {
  const [loading, setLoading] = React.useState(true);
  return (
    <>
      <button onClick={() => setLoading(l => !l)}>Toggle</button>
      <Shimmer loading={loading}>
        <div style={{ padding: 16, background: '#f5f5f5', borderRadius: 8 }}>
          <h3>Title here</h3>
          <p>Description text goes here in this paragraph.</p>
          <button>Action</button>
        </div>
      </Shimmer>
    </>
  );
}
```

If this produces no skeleton rects, the children rendered nothing (empty DOM). Add `dummyData` or use `as` prop.

---
> Source: [jeetvora331/shimmer-trace](https://github.com/jeetvora331/shimmer-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
