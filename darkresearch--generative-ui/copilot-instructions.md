## generative-ui

> This monorepo uses AI-first development. All AI coding tools (Cursor, Claude, GitHub Copilot, etc.) should follow these workflows.

# AI-Assisted Development Workflow

This monorepo uses AI-first development. All AI coding tools (Cursor, Claude, GitHub Copilot, etc.) should follow these workflows.

## Repository Structure

```
generative-ui/
├── apps/
│   ├── debugger/           # Web debugger (port 8081)
│   ├── debugger-ios/       # iOS debugger (port 8082)
│   └── starter/            # Starter template
├── packages/
│   ├── streamdown-rn/      # Streaming markdown renderer
│   ├── galerie-rn/         # Generative UI canvas (experimental)
│   └── debug-components/   # Shared debug components
└── AGENTS.md               # This file
```

**How they relate**:
- Apps use `streamdown-rn` for streaming markdown
- Apps use `galerie-rn` for generative canvas (experimental)
- Debugger apps share components via `debug-components`

## Development Flow

### 1. Working on streamdown-rn

```bash
cd packages/streamdown-rn

# Run tests
bun test

# Watch mode
bun test --watch

# Type check
bun run type-check

# Build
bun run build
```

### 2. Testing with Debugger

```bash
# Terminal 1: Web debugger
cd apps/debugger
bun run dev:web

# Terminal 2: iOS debugger (optional)
cd apps/debugger-ios
bun run dev
```

### 3. Verification Checklist

Before completing any work:
- ✅ All tests pass (`bun test`)
- ✅ Type check passes (`bun run type-check`)
- ✅ Build succeeds (`bun run build`)
- ✅ Manual testing in debugger

## Testing Commands

```bash
# From repo root
bun test              # Run all tests
bun run build         # Build all packages
bun run type-check    # Type check all packages
```

## Package Guidelines

### streamdown-rn

The core streaming markdown renderer. Key areas:

- `src/core/` — Parsing and block splitting logic
- `src/renderers/` — React Native rendering
- `src/themes/` — Theming system
- `src/__tests__/` — Test suites

**When modifying:**
- Add/update tests for new functionality
- Ensure streaming edge cases are handled
- Test with incomplete markdown (simulates real streaming)

### galerie-rn

Experimental canvas for 2D generative layouts. Currently a stub.

### debug-components

Shared components for testing. Add new test components here:

```tsx
// packages/debug-components/src/components/MyComponent.tsx
export const MyComponent = ({ ... }) => { ... };

// packages/debug-components/src/registry.ts
// Register in debugComponentRegistry
```

## Performance Philosophy

Streaming apps render content character-by-character. Performance is critical:

```tsx
// ❌ BAD - Re-renders every character
<Controls streamingText={text} />

// ✅ GOOD - Stable prop
<Controls streamingLength={text.length} />
```

Key principles:
- Block-level memoization (stable blocks don't re-render)
- Character-level parsing for consistent boundaries
- Never pass changing strings as props

## Security

streamdown-rn includes built-in XSS protection:
- URL sanitization (allowlist approach)
- Prop sanitization for custom components
- HTML rendered as text, never executed

See `packages/streamdown-rn/README.md#security` for details.

## AI Development Best Practices

1. **Start with tests** — Write test cases first, implementation second
2. **Use the debugger** — Visual feedback during development
3. **Check streaming edge cases** — Incomplete markdown, chunk boundaries
4. **Run full verification** — Tests, types, build before committing

## Common Tasks

### Adding a Test Preset

Edit `apps/debugger/src/presets.ts`:

```typescript
export const STABLE_PRESETS = {
  my_preset: `# Test\n\nContent here...`,
};
```

### Adding a Debug Component

1. Create component in `packages/debug-components/src/components/`
2. Register in `packages/debug-components/src/registry.ts`
3. Test in debugger with component syntax: `[{c:"MyComponent",p:{...}}]`

### Updating Security Tests

Add tests to `packages/streamdown-rn/src/__tests__/sanitize.test.ts`

---
> Source: [darkresearch/generative-ui](https://github.com/darkresearch/generative-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
