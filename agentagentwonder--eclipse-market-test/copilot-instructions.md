## eclipse-market-test

> **Build/Dev:** `npm run tauri dev` (dev), `npm run tauri build` (production)

# Eclipse Market Pro - Agent Guide

## Commands

**Build/Dev:** `npm run tauri dev` (dev), `npm run tauri build` (production)
**Testing:** `npm run test` (unit), `npm run test:e2e` (Playwright), `npm run test:e2e:ui` (Playwright UI)
**Test single file:** `npm run test -- <file-path>` (e.g., `npm run test -- tests/stocks.test.ts`)
**Lint:** `npm run lint` (check), `npm run lint:fix` (auto-fix)
**Format:** `npm run format` (auto-format), `npm run format:check` (check only)

## Architecture

- **Stack:** Vite + React 18 + TypeScript frontend, Rust/Tauri backend
- **State:** Zustand stores in `src/store/`
- **Backend:** Tauri commands in `src-tauri/src/` (modular: `wallet/`, `market/`, `trading/`, `ai/`, etc.)
- **Database:** SQLite via sqlx, managed in `src-tauri/src/db.rs`
- **Key Modules:** Phantom wallet integration, Solana blockchain, real-time market data, AI sentiment analysis
- **Tests:** Vitest (unit) in `tests/`, Playwright (e2e) in `e2e/`, mobile tests in `mobile-tests/`

## Project Structure

src/
├── components/ # React components
├── store/ # Zustand state stores
├── hooks/ # Custom React hooks
├── utils/ # Utility functions
├── types/ # TypeScript definitions
├── services/ # API services
├── App.tsx # Main app
└── main.tsx # Entry point

src-tauri/src/
├── wallet/ # Wallet operations
├── market/ # Market data
├── trading/ # Trade execution
├── ai/ # AI/sentiment
├── db.rs # SQLite database
└── main.rs # Entry point

tests/ # Vitest unit tests
e2e/ # Playwright e2e tests
mobile-tests/ # Mobile tests


## Code Style

- **Format:** Prettier (semi, singleQuote, printWidth: 100, tabWidth: 2, arrowParens: avoid)
- **TypeScript:** Strict mode, ESNext target, react-jsx (no React import needed)
- **React:** Functional components, hooks recommended
- **Naming:** `camelCase` (JS/TS), `snake_case` (Rust), `PascalCase` (components/types), `UPPER_SNAKE_CASE` (constants)
- **No `any`:** Use proper types; `@typescript-eslint/no-explicit-any` is warn level
- **Unused vars:** Prefix with `_` to ignore (e.g., `_unusedVar`)
- **Error Handling:** Use `anyhow`/`thiserror` in Rust, proper `try/catch` in TypeScript

## Error Analysis Framework

### 1. Syntax & Compilation Errors
- Missing semicolons, brackets, parentheses (Prettier enforces)
- Malformed expressions, invalid operators, illegal tokens
- TypeScript strict mode violations
- Rust compilation errors
- **Check:** Code compiles/builds without errors

### 2. Type System & Compatibility
- `any` types (should be minimal, warn-only)
- Type mismatches between Tauri (Rust) and frontend (TypeScript)
- Missing type definitions, incorrect generics
- Implicit type coercions causing runtime errors
- **Check:** All types explicit, no `any` except necessary

### 3. Tauri Bridge Integrity
- **Critical:** Command name mismatches: `#[tauri::command]` vs `invoke('...')`
- Parameter type mismatches (Rust ↔ TypeScript)
- Async handling (all commands return Promises)
- Serialization errors (serde)
- **Example:**
  ```rust
  #[tauri::command]
  async fn get_balance(address: String) -> Result<f64, String>
const balance = await invoke<number>('get_balance', { address: addr });
4. Import & Dependency Integrity
All imports resolve correctly (check tsconfig paths)
No circular dependencies
Missing deps in package.json or Cargo.toml
Unused imports (ESLint flags)
Check: No import 404s, paths match tsconfig aliases
5. Naming & Reference Errors
Undefined variables, functions, classes, modules
Typos in identifiers
Convention violations (camelCase vs snake_case)
Unused vars not prefixed with _
Check: Naming consistency across files
6. Solana Integration Errors
Wallet adapter config (Phantom)
Transaction signing flow broken
Invalid public key formats
RPC endpoint connectivity issues
Network mismatch (devnet vs mainnet)
Check: Wallet connection flow, transaction handling
7. State Management (Zustand)
Store selectors not typed
Direct state mutation (use set functions)
Subscriptions not cleaned up
Pattern:
interface Store {
  data: Data;
  setData: (data: Data) => void;
}
const useStore = create<Store>(set => ({
  data: {},
  setData: data => set({ data }),
}));
8. Database (SQLite) Errors
SQL syntax in db.rs
sqlx query macro compile-time errors
Connection pool leaks
Schema migration issues
Check: Parameterized queries, no SQL injection
9. React Hooks Rules
Hooks called conditionally or in loops
Hooks not at top level
Dependency arrays incomplete
Fix: Move conditions inside hooks, not around them
10. Logic & Runtime Errors
Null/undefined reference errors
Off-by-one errors, incorrect array indexing
Infinite loops, unreachable code
Unhandled promise rejections
Check: All async properly awaited or .catch()ed
11. Performance Issues
Unnecessary re-renders (use React DevTools Profiler)
Heavy computations in render
Missing memoization (useMemo, useCallback)
WebSocket connection leaks
Check: Bundle size, render performance
12. Security Concerns
Critical: Private keys, API keys in source
Sensitive data in localStorage (use Tauri secure storage)
SQL injection vulnerabilities
XSS in dynamic content
Check: All secrets in environment variables only
13. Documentation & Comments
Typos in comments/
docs

Outdated comments not matching code
Misleading documentation
TODO/FIXME indicating known issues
14. Configuration & Environment
Invalid config files (JSON/TOML syntax)
Missing environment variables
Hardcoded values that should be configurable
Error Output Format
For each issue:

Severity: CRITICAL | HIGH | MEDIUM | LOW
Category: Syntax, Type, Tauri Bridge, Solana, etc.
File Path: Exact location with line:column
Description: Clear explanation
Code Snippet: 5-10 lines of context
Fix Suggestion: Specific, actionable correction
Impact: What breaks if not fixed
Example:

[HIGH] Type Mismatch - Tauri Bridge
File: src/services/wallet.ts:24:15
Description: invoke() expects string, got number
Code:
  23: async function getBalance(address: string) {
  24:   const balance = await invoke('get_balance', { address: 123 });
  25:   return balance;
Fix: Change to { address: address } (pass string)
Impact: Runtime error on wallet balance call
Priority Matrix
CRITICAL (Fix Immediately):

Build/compile failures, syntax errors, missing deps
Security vulnerabilities (exposed keys, SQL injection)
Tauri bridge complete breakage
HIGH (Fix Soon):

Runtime crashes, type errors blocking features
Wallet/Solana integration errors
Database connection failures
Broken core features
MEDIUM (Schedule):

ESLint warnings, deprecated APIs
Performance bottlenecks
Missing error handling
Incomplete types
LOW (Nice to Have):

Style inconsistencies (Prettier auto-fixes)
Comment typos
Minor optimizations
Unused vars already prefixed with _
Common Fix Patterns
Tauri Bridge:

// Match Rust command name and params exactly
#[tauri::command]
async fn get_balance(address: String) -> Result<f64, String>

const balance = await invoke<number>('get_balance', { address: addr });
Zustand Types:

interface AppStore {
  user: User | null;
  setUser: (user: User | null) => void;
}
const useAppStore = create<AppStore>(set => ({ /* ... */ }));
Unused Vars:

const handleClick = (_event: React.MouseEvent) => { /* ... */ };
SQL Safety:

sqlx::query!("SELECT * FROM users WHERE id = ?", user_id)
Best Practices
DO:

Run npm run lint:fix && npm run format before commit
Use TypeScript strict mode (enabled)
Handle all async errors with try/catch
Prefix unused vars with _
Use Zustand for shared state
Write tests for new features
Follow naming conventions strictly
DON'T:

Commit with linting errors
Use any type (unless necessary, warn-only)
Mutate Zustand state directly
Skip error handling on Tauri commands
Hardcode secrets/API keys
Use class components (prefer functional + hooks)
Mix naming conventions
Key Dependencies
Frontend: @tauri-apps/api, zustand, @solana/web3.js, @solana/wallet-adapter-react, framer-motion, lucide-react
Backend: tauri, sqlx, anyhow, thiserror, serde, tokio
Testing: vitest, @playwright/test, @testing-library/react

Testing Standards
Unit (Vitest): tests/*.test.ts, run npm run test
E2E (Playwright): e2e/, run npm run test:e2e or npm run test:e2e:ui
Mobile: mobile-tests/
Coverage: Utils, stores, hooks (unit), critical flows (e2e)
Agent Tips
Context first: Understand feature purpose before fixing
Consistency: Match existing patterns
Test impact: Consider downstream effects
Security: Never compromise for convenience
Verify: Ensure fixes don't break other features
Document: Comment non-obvious fixes
Ask: Flag ambiguous issues for human review
Future Evolution
AR/VR trading interfaces
Advanced behavioral coaching
Developer plugin platform
Cross-chain support (beyond Solana)
Institutional-grade analytics
Multi-account management
Keep scalability, extensibility, maintainability in mind.

Version: 2.0 | Stack: React 18 + TypeScript + Vite + Tauri + Rust + SQLite + Solana

---
> Source: [AgentAgentwonder/Eclipse-Market-test](https://github.com/AgentAgentwonder/Eclipse-Market-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
