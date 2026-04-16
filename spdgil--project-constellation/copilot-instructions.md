## project-constellation

> Keep React components small and focused


# Component Size Guidelines

## Size Limits

- **Components**: Aim for under 200 lines, refactor at 400+ lines
- **Hooks**: Extract reusable logic into custom hooks in `lib/hooks/`
- **Files**: One primary component per file

## When to Split

Split a component when it:
- Has multiple distinct UI sections (e.g. sidebar + canvas + bottom sheet)
- Contains complex state logic that could be a hook
- Renders different things based on mode/state
- Is hard to understand at a glance

## Refactoring Patterns

```tsx
// ❌ BAD — God component
function MapView() {
  // 50 lines of state and hooks
  // 100 lines of handlers
  // 300 lines of JSX with sidebar, map, drawer, bottom sheet
}

// ✅ GOOD — Composed from focused components
function MapView() {
  return (
    <div className="flex flex-1 min-h-0">
      <LgaPanel />
      <div className="flex-1 flex flex-col">
        <MapCanvas />
        <LgaBottomSheet />
      </div>
      <DealDrawer />
    </div>
  );
}
```

## Extract Custom Hooks

```tsx
// ❌ BAD — Logic mixed with rendering
function DealsSearch({ baseDeals }) {
  const [deals, setDeals] = useState(baseDeals);
  useEffect(() => {
    const saved = loadAllDeals();
    // 15 lines of merge logic...
  }, [baseDeals]);
}

// ✅ GOOD — Props used directly, no wrapper hook needed
function DealsSearch({ deals }) {
  // Clean rendering logic
}
```

## Colocation

Keep related code together:
- `ComponentName.tsx` — main component
- `ComponentName.test.tsx` — tests (always colocated)
- Custom hooks go in `lib/hooks/` when shared across components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spdgil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
