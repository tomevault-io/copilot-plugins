---
name: react-class-to-functional
description: Convert React class components to functional components with hooks
argument-hint: <file-path>
---

# React Class to Functional Component Converter

Convert the React class component at `$ARGUMENTS` to a functional component using React hooks.

## Conversion Rules

### State
- `this.state = { ... }` → individual `useState` hooks
- `this.setState({ key: value })` → setter function
- `this.setState(prev => ...)` → functional update form
- Mutable objects (Set/Map) must be cloned on update:
  ```typescript
  setExpandedRows(prev => { const next = new Set(prev); next.add(key); return next; });
  ```

### Lifecycle → useEffect

| Class | Functional |
|-------|-----------|
| `componentDidMount` | `useEffect(fn, [])` |
| `componentDidUpdate` | `useEffect(fn)` or `useEffect(fn, [deps])` |
| `componentWillUnmount` | `useEffect(() => cleanup, [])` |
| mount + unmount | single `useEffect` with cleanup return |

### Other Mappings
- `React.createRef()` → `useRef(null)`
- `static contextType` → `useContext(MyContext)`
- Bound methods / arrow class methods → local function inside functional component
- Non-reactive instance variables (`this.timer`) → `useRef`
- `createSelector` (reselect) → `useMemo` with explicit dependency array

### Props and Defaults
- `this.props.x` → destructured props
- `static defaultProps` → default parameter values
- **Critical**: mark defaultProps as optional (`?`) in the interface:
  ```typescript
  interface Props { sortSetting?: SortSetting; }
  function Component({ sortSetting = defaultValue }: Props) { ... }
  ```

### setState with Callback
`this.setState(update, callback)` has no direct hook equivalent. Use ref + useEffect:
```typescript
const pendingRef = useRef(false);
const prevRef = useRef(value);

const onUpdate = useCallback(() => {
  pendingRef.current = true;
  setValue(newValue);
}, []);

useEffect(() => {
  if (pendingRef.current && prevRef.current !== value) {
    pendingRef.current = false;
    doCallback();
  }
  prevRef.current = value;
}, [value, doCallback]);
```

## Generic Components

- `class MyTable extends SortedTable<Row> {}` → `const MyTable = SortedTable<Row>;`
- Exported type aliases require the props interface to also be exported (else "cannot be named" build errors)
- TypeScript may fail to infer generics with spread props — add explicit type params: `<SortedTable<Row> {...props} />`

## Workflow

1. Read the file and summarize: state vars, lifecycle methods, refs, methods, generics, files extending this class
2. Before converting each method, verify it's actually called — delete dead code, don't convert it
3. Write the converted component, preserving imports/comments/exports and adding hook imports
4. Search for and update dependent files that extend the class

## Verification Checklist

- [ ] All `this.` references removed
- [ ] Hooks follow Rules of Hooks (top level, consistent order)
- [ ] TypeScript types preserved; exports unchanged
- [ ] defaultProps marked optional in interface
- [ ] Exported type aliases have their props interfaces exported
- [ ] Files extending the class updated to type alias pattern
- [ ] No dead code converted (every `useCallback` is actually called)
- [ ] No orphaned helpers (if logic was inlined, delete the original)
- [ ] No unused destructured props
- [ ] No failed intermediate attempts left behind
- [ ] Within the appropriate directory with `package.json`, run ESLint (fix) and Tests

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/cockroachdb/cockroach)
<!-- tomevault:2.0:skill_md:2026-04-05 -->
