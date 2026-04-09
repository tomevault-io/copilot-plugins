## tipbox-app

> - "Server State" = API/backend kaynaklı veri (listeler, detaylar, yorumlar, mesajlar, bildirimler).

# Tipbox State Management Rules (React Query + Zustand + Feature Slices)

## 0) Terminology
- "Server State" = API/backend kaynaklı veri (listeler, detaylar, yorumlar, mesajlar, bildirimler).
- "Client State" = UI state, session state, flow context, local draft, device/app config.

---

## 1) React Query is mandatory for Server State
### 1.1 Store (Zustand) MUST NOT hold server lists or server objects as source of truth
- Do NOT store: posts[], brands[], events[], notifications[], messages[], marketplaceItems[] in Zustand.
- These MUST be fetched and owned by React Query.

### 1.2 React Query responsibilities
- Fetching, loading/error state, retry/backoff.
- Deduplication and caching.
- Pagination/infinite queries.
- Mutations + invalidation (or optimistic updates if appropriate).

---

## 2) Zustand is mandatory for Client State
### 2.1 Zustand SHOULD hold:
- Auth/session: accessToken, refreshToken, sessionStatus, minimal `me`.
- App config: themeMode, language, network flags, deepLink params.
- UI state: activeTab, filters, sortBy, modal/bottomsheet open/close, input drafts.
- Flow context: cross-screen navigation context (e.g., Catalog -> CreatePost).
- Realtime flags: socketConnected, typing state (scoped properly).

### 2.2 Zustand SHOULD hold IDs instead of full objects
- Prefer: selectedProductGroupId, selectedBrandId, selectedEventId
- Avoid: selectedProductGroup object, selectedBrand object unless it is a minimal snapshot for UI and short-lived.

---

## 3) Global Store vs Feature Store decision
### 3.1 Place in GLOBAL store (src/store/*) if:
- Used across multiple features OR app-wide concerns:
  - auth/session
  - app config (theme/language/network)
  - global modal/toast
  - global navigation tracking (optional)

### 3.2 Place in FEATURE store (src/features/<feature>/store/*) if:
- Only affects that feature’s UI and decisions:
  - feed filters/tab/scroll state
  - catalog selection UI and sorting UI
  - events tab and local RSVP UI
  - inbox drafts/typing UI state

### 3.3 Flow state MUST live in the flow owner feature
- For Catalog -> Create Post:
  - CreatePostFlowStore MUST live in `src/features/post/store/`.
  - It may store selectedProductGroupId + optional minimal snapshot.

---

## 4) Slice approach requirements
### 4.1 Use slices to avoid mega-store coupling
- Global slices: auth.slice, appConfig.slice, modal.slice
- Feature stores: useFeedUIStore, useCatalogUIStore, useEventsUIStore, useInboxUIStore
- Flow store: useCreatePostFlowStore

### 4.2 Flow snapshot rules (if used)
- Snapshot must be minimal (only fields required for immediate UI).
- Snapshot must NOT be persisted.
- Snapshot must have TTL (expiresAt) and be cleared via:
  - soft-GC on access OR
  - lifecycle cleanup on cancel/submit/back.

---

## 5) Zustand selector rules (performance critical)
### 5.1 NEVER subscribe to entire store
- Do NOT: const store = useStore()
- MUST: const value = useStore(s => s.value)

### 5.2 Use shallow when selecting multiple fields
- Use `shallow` for tuples or small objects to minimize re-renders.

### 5.3 Avoid frequently-updating values in global store
- Scroll offsets, animation-driven values, high-frequency signals should stay local when possible.
- If necessary, throttle/debounce updates.

---

## 6) React Query caching rules
### 6.1 Cache SHOULD be used for:
- Brand detail, product group metadata, event detail, catalog browsing pages.
- Any data that is read frequently and changes relatively infrequently.

### 6.2 Cache SHOULD be limited for:
- Highly dynamic feeds (use shorter staleTime).
- Queries producing many filter combinations (avoid cache explosion).

### 6.3 QueryKey rules (avoid cache explosion)
- Query keys MUST be deterministic and normalized.
- Do not include unstable objects directly unless normalized (stable shape + sorted keys).
- Only include fields that change backend result.

### 6.4 Invalidation rules
- On mutations, invalidate only related queries.
- Do NOT blanket invalidate all queries unless required (e.g., logout).

---

## 7) Memory rules (avoid double buffering)
- Never keep the same server dataset in both React Query cache and Zustand.
- If a flow needs instant UI, store only:
  - id + minimal snapshot + TTL
  - and always treat React Query as source of truth.

---

## 8) Logout / Session change rules
- On logout:
  - Clear auth/session slices.
  - Clear flow stores.
  - Remove React Query caches that are user-specific (queryClient.clear or targeted remove).
- On login:
  - Rehydrate session, then allow queries.

---

## 9) Implementation checklist (per feature)
- Feature UI state: feature store.
- Server data: React Query hook with normalized queryKey.
- Component reads only needed store fields via selectors.
- Derived lists computed in component (useMemo), not stored globally.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tipboxco)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tipboxco)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
