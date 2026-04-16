## pets

> React and Next.js 15 patterns and common pitfalls


# React & Next.js 15 Best Practices

## Next.js 15 Breaking Changes

### 1. Dynamic Route Params (CRITICAL)
**Next.js 15 made `params` readonly and requires null checks.**

❌ **BAD** (will crash):
```typescript
export default function Page() {
  const params = useParams();
  const id = params.id as string; // Error: Cannot assign to readonly property
}
```

✅ **GOOD**:
```typescript
export default function Page() {
  const params = useParams();
  const id = (params?.id as string) || '';
}
```

**Apply to ALL dynamic routes:**
- `[matchId]/page.tsx`
- `[roomId]/page.tsx`
- `[petId]/page.tsx`
- etc.

### 2. Leaflet Map Re-initialization

**Problem**: `Map container is already initialized`

❌ **BAD**:
```tsx
<MapContainer center={center} zoom={13}>
```

✅ **GOOD** (add stable keys):
```tsx
<div key="map-container">
  <MapContainer center={center} zoom={13} key="leaflet-map">
```

### 3. React Hooks with Side Effects

**Problem**: Hooks causing infinite re-renders

❌ **BAD**:
```typescript
export function useWebSocket(userId?: string) {
  useEffect(() => {
    const socket = connect(userId); // May trigger re-renders
    return () => disconnect();
  }, [userId]);
}
```

✅ **GOOD** (stub unimplemented features):
```typescript
export function useWebSocket(userId?: string) {
  useEffect(() => {
    if (!userId) return;
    console.warn('[WebSocket] Not yet implemented');
    // Actual implementation later
  }, [userId]);
}
```

## Error Handling

### Geolocation API
Always provide fallback locations:

```typescript
navigator.geolocation.watchPosition(
  (position) => {
    handleLocation(position);
  },
  (error) => {
    // Don't just log - provide fallback!
    if (error.code === error.PERMISSION_DENIED) {
      console.warn('📍 Geolocation: Permission denied');
    }
    // Fallback to default location
    handleLocation({ lat: 40.7128, lng: -74.0060 });
  }
);
```

## Component Patterns

### Suspense Boundaries
Always wrap pages using `useSearchParams()`:

```tsx
function PageContent() {
  const searchParams = useSearchParams();
  // ... component logic
}

export default function Page() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <PageContent />
    </Suspense>
  );
}
```

## Performance

### Avoid Re-renders
- Use `memo()` for expensive components
- Add stable `key` props to lists and maps
- Use `useCallback` for functions passed as props
- Use `useMemo` for expensive calculations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openann19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
