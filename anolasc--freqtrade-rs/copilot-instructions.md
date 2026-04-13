## freqtrade-rs

> This file provides development guidelines for agentic coding agents working on freqtrade-rs.

# AGENTS.md - Guide for Agentic Coding Agents

This file provides development guidelines for agentic coding agents working on freqtrade-rs.

## Project Overview

freqtrade-rs is a Rust + Tauri desktop crypto trading bot with a React 19 frontend. Backend uses async/await with Tokio, database is SQLite with SQLx.

## Build/Lint/Test Commands

### Backend (Rust)

```bash
cd src-tauri

# Check compilation (fast, no build)
cargo check

# Build debug
cargo build

# Build release
cargo build --release

# Run dev server
cargo run

# Run all tests
cargo test

# Run single test
cargo test test_name          # By function name
cargo test module_name        # By module name

# Linting
cargo clippy

# Formatting
cargo fmt                     # Auto-format
cargo fmt -- --check          # Verify formatting

# All checks (CI)
cargo fmt -- --check && cargo clippy && cargo test
```

### Frontend (React + TypeScript)

```bash
cd src

# Install dependencies
pnpm install

# Dev server
pnpm run dev

# Production build
pnpm run build

# Linting (errors and unused imports)
pnpm run lint

# Type checking
pnpm run type-check

# Auto-format
pnpm run format

# Check formatting
pnpm run format:check

# Run tests with coverage
pnpm run test

# Run single test file
pnpm run test -- testFileName

# Preview build
pnpm run preview
```

### Full Stack

```bash
# Run both frontend + backend dev
pnpm run tauri:dev

# Build production bundle
pnpm run tauri:build
```

## Code Style Guidelines

### Rust Conventions

**Formatting (rustfmt.toml)**
- Edition: 2024
- Max width: 120 characters
- Tab spaces: 4
- Newline style: Unix
- Reorder imports: enabled

**Error Handling**
- Use `crate::error::AppError` enum for all errors
- Define Result type: `pub type Result<T> = std::result::Result<T, AppError>;`
- Use `?` operator for propagation
- Implement `From` traits for error conversions
- Never suppress errors with `as any`, `@ts-ignore`, `@ts-expect-error`

**Concurrency**
- Use `Arc<Mutex<T>>` or `Arc<RwLock<T>>` for shared state
- Always use async/await with Tokio (never block)
- All I/O operations must be non-blocking

**Money/Financial Values**
- Use `rust_decimal::Decimal` for all financial calculations
- Never use floating-point for money

**DateTime**
- All timestamps use `chrono::DateTime<Utc>`

**Naming**
- Structs/Praits/Enums: PascalCase (e.g., `TradingBot`, `AppError`)
- Functions/Variables: snake_case (e.g., `start_bot`, `get_status`)
- Constants: SCREAMING_SNAKE_CASE
- Modules: snake_case

**Imports**
- Group std imports first, then external, then crate
- Use `crate::` prefix for internal imports
- Prefer absolute paths over relative (`crate::module` over `super::module`)

**Module Pattern**
```
module/
├── mod.rs          # Module entry, exports
├── module.rs       # Main implementation
├── submodule1.rs   # Sub-components
└── submodule2.rs
```

**Tauri Commands**
```rust
#[tauri::command]
pub async fn command_name(state: State<'_, AppState>) -> Result<ReturnType> {
    // Implementation
}
```

### TypeScript/React Conventions

**TypeScript Setup**
- Version: 5.6
- React: 18.3
- Strict mode: enabled

**ESLint Rules**
- `no-unused-vars`: warn (except `_` prefix)
- `unused-imports/no-unused-imports`: error
- `@typescript-eslint/no-explicit-any`: off (permissive)
- React hooks exhaustive deps: off

**Imports**
- Organize: React imports → external → internal
- Use `@/` alias for `src/` (configured in vite.config.ts)
- Example: `import { useState } from 'react';`
- Example: `import { getTrade } from '@/services/api';`
- Remove unused imports (eslint will error)

**State Management**
- Use Zustand for global state (see `stores/appStore.ts`)
- Pattern: `create<AppState>((set) => ({ ... }))`
- Actions grouped in `actions` object

**Components**
- Use TypeScript interfaces for props
- Name components PascalCase (`TradeRow.tsx`)
- Use shadcn/ui pattern for base components (`ui/card.tsx`, `ui/button.tsx`)
- Functional components with hooks

**Styling**
- TailwindCSS for all styling
- Use `cn()` utility for class merging (`lib/utils.ts`)
- Follow shadcn/ui color patterns (primary, secondary, muted, destructive)

**Naming**
- Components/Functions/Types: PascalCase
- Variables/constants: camelCase
- Files: kebab-case (`trade-row.tsx`, not `TradeRow.tsx`)
- Interfaces: PascalCase with `I` prefix optional (`Trade`, not `ITrade`)

**Async/Await**
- Always handle errors with try/catch
- Log errors with `console.error()`
- Never leave empty catch blocks

**Utility Functions**
- Place in `lib/utils.ts`
- Export named functions
- Follow existing patterns: `formatNumber()`, `formatCurrency()`, `formatDate()`

### General Rules

**NEVER:**
- Suppress type errors with `as any`, `@ts-ignore`, `@ts-expect-error`
- Leave empty catch blocks
- Block on async operations in Rust
- Use floating-point for money/financial values
- Create commit without explicit user request

**ALWAYS:**
- Use `?` for error propagation in Rust
- Handle async errors with try/catch in TypeScript
- Run `cargo fmt` before committing Rust code
- Run `pnpm run lint` before committing TypeScript code
- Verify diagnostics pass (`lsp_diagnostics`) before completing tasks

## Key File Locations

| Purpose | Location |
|---------|----------|
| Rust error types | `src-tauri/src/error.rs` |
| Rust types | `src-tauri/src/types.rs` |
| TypeScript types | `src/types/index.ts` |
| Rust formatting | `src-tauri/rustfmt.toml` |
| Rust toolchain | `src-tauri/rust-toolchain.toml` |
| ESLint config | `src/eslint.config.js` |
| Tailwind config | `src/tailwind.config.js` |
| Vite config | `src/vite.config.ts` |
| Tauri commands | `src-tauri/src/commands.rs` |

## Common Patterns

**Rust Database Access**
```rust
// All DB through Repository pattern
repository.get_open_trades().await?
```

**Rust Shared State**
```rust
Arc::new(Mutex::new(data))  // For exclusive access
Arc::new(RwLock::new(data)) // For read-heavy access
```

**TypeScript API Calls**
```typescript
// Via Tauri invoke
import { invoke } from '@tauri-apps/api/core';
return invoke('command_name', { param });
```

**TypeScript Component**
```typescript
import { useEffect, useState } from 'react';
import { useAppStore } from '@/stores/appStore';

export function ComponentName() {
  const { actions } = useAppStore();
  const [state, setState] = useState<Type>();

  useEffect(() => { /* ... */ }, []);

  return <div>{/* JSX */}</div>;
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ANOLASC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
