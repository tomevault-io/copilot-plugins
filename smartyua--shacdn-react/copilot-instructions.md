## shacdn-react

> 1. ✅ Run `npm run lint` after making ANY code changes


# 🚨 CRITICAL: Mandatory Lint Verification

## REQUIRED After Every Code Change

**YOU MUST ALWAYS:**

1. ✅ Run `npm run lint` after making ANY code changes
2. ✅ Fix ALL lint errors before completing the task
3. ✅ Verify 0 errors, 0 warnings
4. ✅ Never consider a task complete with lint errors

## Lint Check Command

```bash
npm run lint
```

**Expected output:**
```
✔ No errors found
```

## If Lint Fails

1. Read the error messages carefully
2. Fix each error reported
3. Run `npm run lint` again
4. Repeat until 0 errors

## Common Lint Errors and Fixes

### Unused Variables
```typescript
// ❌ BAD
const [value, setValue] = useState(0);
// setValue never used

// ✅ GOOD - Remove unused variable
const [value] = useState(0);
// Or use it
const [value, setValue] = useState(0);
setValue(5);
```

### Missing Dependencies in useEffect
```typescript
// ❌ BAD
useEffect(() => {
  doSomething(value);
}, []); // value not in dependency array

// ✅ GOOD
useEffect(() => {
  doSomething(value);
}, [value]);
```

### Any Type Usage
```typescript
// ❌ BAD
function process(data: any) {}

// ✅ GOOD
function process(data: SomeType) {}
```

### React Hooks Rules
```typescript
// ❌ BAD - Calling setState in useEffect synchronously
useEffect(() => {
  setState(value);
}, []);

// ✅ GOOD - Use initial state
const [state, setState] = useState(() => getValue());
```

## Build Verification

After lint passes, also verify build:

```bash
npm run build
```

Should complete without TypeScript errors.

## Workflow Checklist

- [ ] Make code changes
- [ ] Run `npm run lint`
- [ ] Fix any errors
- [ ] Run `npm run lint` again (verify 0 errors)
- [ ] Test in browser (optional)
- [ ] Run `npm run build` (optional, for major changes)
- [ ] ✅ Task complete ONLY when lint passes

## ESLint Configuration

Project uses:
- `@typescript-eslint/eslint-plugin`
- `eslint-plugin-react-hooks`
- `eslint-plugin-react-refresh`

Strict rules for:
- Type safety
- React hooks
- Unused code
- Code quality

## Never Ignore Lint

❌ DO NOT:
- Use `// eslint-disable-next-line` without good reason
- Commit code with lint errors
- Skip lint checks
- Consider task done with errors

✅ ALWAYS:
- Fix the underlying issue
- Keep code clean
- Maintain 0 lint errors
- Verify before completing

## Automated Checks

This project enforces quality through:
1. ESLint for code quality
2. TypeScript for type safety
3. Build process for compilation errors

ALL must pass before task completion.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smartyua)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/smartyua)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
