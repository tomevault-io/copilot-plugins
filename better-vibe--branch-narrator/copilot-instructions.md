## branch-narrator

> Project conventions and architecture rules for branch-narrator


# branch-narrator Project Rules

## Project Overview

**branch-narrator** is a heuristics-only CLI that analyzes git diffs and generates PR descriptions.

### Core Principles

1. **No LLM/AI calls** - Deterministic, local-only analysis
2. **No network calls** - Works fully offline
3. **Evidence-based** - Never invent "why", only report what changed
4. **Extensible** - Profile-based analyzer architecture

## Key Documentation

Read these before making changes:

- `docs/README.md` - Project overview
- `docs/architecture.md` - System design and data flow
- `docs/analyzers.md` - How analyzers work
- `docs/types.md` - All TypeScript types

## Architecture

```
CLI → Git Collector → ChangeSet → Profile → Analyzers → Findings → Renderer → Output
```

### Directory Structure

| Directory | Purpose |
|-----------|---------|
| `src/core/` | Types, errors, utilities |
| `src/git/` | Git command execution and parsing |
| `src/analyzers/` | Individual analyzers (one per file) |
| `src/profiles/` | Profile definitions |
| `src/render/` | Markdown and JSON output |
| `tests/` | Vitest tests with fixtures |
| `docs/` | Comprehensive documentation |

## Coding Conventions

### TypeScript

- Use explicit types for function parameters and returns
- Prefer `interface` over `type` for object shapes
- Use discriminated unions for findings (`type` field as discriminator)
- Export types from `src/core/types.ts`

### Naming

| Element | Convention | Example |
|---------|------------|---------|
| Files | kebab-case | `route-detector.ts` |
| Types | PascalCase | `RouteChangeFinding` |
| Functions | camelCase | `detectMethods()` |
| Constants | UPPER_SNAKE | `CRITICAL_PACKAGES` |
| Analyzers | camelCase + Analyzer | `routeDetectorAnalyzer` |

### Imports

- Use `.js` extension for local imports (ESM)
- Group imports: external, then internal by path
- Export from index files for public API

## Adding Features

### New Analyzer Checklist

1. Create `src/analyzers/{name}.ts`
2. Implement `Analyzer` interface
3. Add finding type to `src/core/types.ts`
4. Export from `src/analyzers/index.ts`
5. Add to profiles in `src/profiles/`
6. Update `src/render/markdown.ts` if needed
7. Write tests in `tests/{name}.test.ts`
8. Update `docs/analyzers.md`
9. Update `docs/changelog.md`

### New Profile Checklist

1. Create `src/profiles/{name}.ts`
2. Add to `ProfileName` type
3. Add detection logic to `detectProfile()`
4. Export from `src/profiles/index.ts`
5. Update CLI help text
6. Update `docs/profiles.md`

## Testing

### Run Tests

```bash
bun test              # All tests
bun test {file}       # Single file
bun run test:watch    # Watch mode
```

### Test Conventions

- Use fixtures from `tests/fixtures/index.ts`
- Test file names match source: `{name}.test.ts`
- Group tests with `describe()` by function/feature
- Use `createChangeSet()` and `createFileDiff()` helpers

## Building

```bash
bun run build         # Production build
bun run dev           # Watch mode
bun run typecheck     # Type checking only
```

## Package Manager

**Always use bun** as the package manager:

```bash
bun install           # Install deps
bun add {pkg}         # Add dependency
bun add -D {pkg}      # Add dev dependency
bun test              # Run tests
bun run build         # Build
```

## Git Conventions

- Commit messages: `type: description`
- Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
- Update `docs/changelog.md` for user-facing changes

## Common Patterns

### Analyzer Pattern

```typescript
import type { Analyzer, ChangeSet, Finding } from "../core/types.js";

export const myAnalyzer: Analyzer = {
  name: "my-analyzer",
  analyze(changeSet: ChangeSet): Finding[] {
    const findings: Finding[] = [];

    for (const file of changeSet.files) {
      // Detection logic
    }

    return findings;
  },
};
```

### Finding Type Pattern

```typescript
export interface MyFinding {
  type: "my-finding";  // Discriminator
  // ... fields
}

// Add to union
export type Finding =
  | FileSummaryFinding
  // ... existing
  | MyFinding;
```

### Risk Flag Pattern

```typescript
if (riskyCondition) {
  findings.push({
    type: "risk-flag",
    risk: "high" | "medium" | "low",
    evidence: "Human-readable explanation",
  });
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/better-vibe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
