## easysoft

> Ask the user which tracker to use for each workflow; local Markdown under `.scratch/` is supported. See `docs/agents/issue-tracker.md`.

# AGENTS.md - POS System Development Guidelines

## Agent skills

### Issue tracker

Ask the user which tracker to use for each workflow; local Markdown under `.scratch/` is supported. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the standard five-role triage vocabulary. See `docs/agents/triage-labels.md`.

### Domain docs

This is a single-context repository. See `docs/agents/domain.md`.

## Build, Lint, and Test Commands

### Development
```bash
npm run dev              # Start dev server (http://localhost:5173)
npm run dev:https        # Start with HTTPS
npm run dev:network      # Start accessible on network
npm run electron:dev     # Run with Electron (React + electron)
npm run electron:dev-debug  # Run with Electron + Chrome debugging
```

### Building
```bash
npm run build            # Build for production (outputs to dist/)
npm run preview          # Preview production build locally
```

### Linting & Type Checking
```bash
npm run lint             # Run ESLint on all files
```

### Testing
```bash
npm test                 # Run all tests
npm test -- src/tests/Auth.test.tsx  # Run specific test file
npm test -- --run       # Run tests once (no watch mode)
npm test -- --coverage  # Run with coverage report
```

### End-to-end (Playwright)
```bash
npx playwright install chromium   # One-time browser download (per machine)
npm run test:e2e                  # Headless E2E (starts Vite via playwright.config)
npm run test:e2e:ui               # Playwright UI mode
npm run test:e2e:headed           # See the browser while tests run
npm run test:e2e:debug            # Step through with Playwright Inspector
```

`e2e/auth.spec.ts` signs in as the bootstrap admin from `public/bootstrap-data.json` using the demo password **`password`** (SHA-256 stored in `password_hash`). Adjust that file if you change credentials.

### Electron Distribution
```bash
npm run electron:dist           # Build for current platform
npm run electron:dist:all       # Build for all platforms
npm run electron:pack           # Build and package
```

---

## Code Style Guidelines

### TypeScript Conventions

**Interface Naming:**
- Entities: `PascalCase` (e.g., `Employee`, `Product`)
- Props: `ComponentNameProps` (e.g., `LoginFormProps`)
- Context: `ContextNameType` (e.g., `AuthContextType`)

**Type Rules:**
- **NEVER** use `any` - define proper interfaces
- Use `Pick<T, K>` for partial interfaces
- Use unions for controlled values: `type Status = 'active' | 'inactive'`
- Enable strict mode - `noUnusedLocals`, `noUnusedParameters` enforced

### Component Structure (Enforced Order)

```typescript
const ComponentName: React.FC<ComponentNameProps> = ({ prop1, prop2 }) => {
  // 1. Hooks (useState, useEffect, useContext)
  const [state, setState] = useState(initialValue);
  const { globalState } = useContext();

  // 2. Event handlers
  const handleClick = () => { /* ... */ };

  // 3. Computed values (useMemo)
  const computed = useMemo(() => expensiveCalc(state), [state]);

  // 4. Effects (useEffect)
  useEffect(() => { /* ... */ }, [dependencies]);

  // 5. Render
  return <div>...</div>;
};
```

### Import Order

1. React hooks (`React`, `useState`, `useEffect`)
2. External libraries (`lucide-react` icons)
3. Local contexts (`../../contexts/`)
4. Local components (`../components/`)
5. Types (`../../types/`)
6. Utils (`../../utils/`)

### File Organization

```
src/
├── components/
│   ├── Auth/           # Feature-specific folders
│   ├── Layout/
│   └── VirtualKeyboard.tsx  # Shared at root
├── contexts/           # React contexts
├── pages/             # Page components
├── types/             # TypeScript interfaces
├── services/          # API/service layers
├── hooks/             # Custom hooks
└── utils/             # Helper functions
```

### React Patterns

**State Management:**
- `useState`: Component-specific UI state
- `useReducer`: Complex state logic
- Context: Global app state (auth, cart)
- `useMemo`/`useCallback`: Performance optimization

**Component Patterns:**
- Use `React.memo` for components with frequent re-renders
- Memoize expensive calculations with `useMemo`
- Memoize handlers passed to children with `useCallback`

### Error Handling

```typescript
// Async error handling pattern
try {
  const result = await asyncOperation();
  setState({ data: result, loading: false, error: null });
} catch (error) {
  setState({ 
    data: null, 
    loading: false, 
    error: error instanceof Error ? error.message : 'Unknown error' 
  });
}

// UI Error display
{error && (
  <div className="bg-red-50 border-2 border-red-200 rounded-2xl p-6">
    <p className="text-red-700 text-xl font-medium">{error}</p>
  </div>
)}
```

### Tailwind CSS Guidelines

**Touch Screen Optimization (CRITICAL):**
- Minimum touch targets: `min-h-touch` (`3.75rem`, ~60px at 16px root); secondary `min-h-touch-sm`; compact `min-h-touch-xs` (~44px)
- Large action buttons: `min-h-20` (~5rem)
- Employee cards: `min-h-70` (~17.5rem tall target at default root)
- Avoid dropdowns and hover-dependent interactions

**Spacing Scale:**
- `gap-2` (8px): Tight spacing (keyboard keys)
- `gap-4` (16px): Standard spacing
- `gap-6` (24px): Comfortable spacing
- `gap-8` (32px): Large spacing (cards)

**Transitions:**
- `duration-200`: Buttons, hover effects
- `duration-300`: Cards, larger elements
- `duration-150`: Keyboard keys (fast response)

**Role-Based Colors (MANDATORY):**
- Admin: `from-red-500 to-pink-600`
- Manager: `from-orange-500 to-amber-600`
- Cashier: `from-blue-500 to-purple-600`

**Functional Colors:**
- Success: `bg-green-500 hover:bg-green-600`
- Danger: `bg-red-500 hover:bg-red-600`
- Warning: `bg-orange-500 hover:bg-orange-600`

---

## Testing Patterns

**Test File Location:** `tests/` (root level, not inside src/)

**Test Structure:**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';

describe('ComponentName', () => {
  const mockProps = { /* ... */ };

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('renders correctly', () => {
    render(<Component {...mockProps} />);
    expect(screen.getByText('Expected')).toBeInTheDocument();
  });
});
```

**Mocking Services:**
```typescript
vi.mock('../src/services/employeeService', () => ({
  employeeService: {
    getEmployeeByNumber: vi.fn(),
    // ...
  }
}));
```

### Fiscal signing (AT / RSG)

- **Browser / dev:** PEM PKCS#8 in Settings is imported with Web Crypto (`WebCryptoRsaSha1Signer`) in the renderer.
- **Electron:** Prefer **Guardar chave no armazenamento seguro** (Settings): PEM is encrypted with `safeStorage` in the main process; signing runs over IPC (`fiscal:sign-hash-plaintext`) via `ElectronSafeStorageSigner`, so the private key is not kept in renderer memory or localStorage. If a secure key exists, `createSignerFromSettings` uses it before falling back to the PEM field.
- **Key rotation:** After installing a new private key, an **admin** uses **Definições → Fiscal AT → Registar rotação de chave (HashControl +1)**. This bumps `fiscal.hashControlVersion` for **new** documents and appends a `KEY_ROTATED` row to the fiscal audit log (`/fiscal-audit`). Issued documents keep their stored `hash_control`.

### Fiscal & transaction data (AT posture)

**What is the “source of truth”?**

- **Fiscal sales (FT/FS, etc.):** The **till** is the **first writer** — it must work **offline** and records the **hash chain + signature** and local fiscal rows (Dexie). The **integrity** of a document is the **cryptographic chain**, not an editable row.
- **Server (Supabase):** Treat as a **replica and archive** for sync, reporting, and backup. It should **not** allow rewriting **sealed** fiscal documents. Conflicts on finalized docs are invalid: do not “merge by editing” — only **new** documents (e.g. **NC**).
- **Master data (employees, products, categories):** The **server** is the usual **source of truth** for identity and catalog; those rows are not the AT invoice chain. They still need **RLS, auth, and least privilege** (separate concern from hash integrity).

**Edge cases**

- **Offline:** New sales are recorded **only** on device until sync; the app does not require the server to finalize fiscal checkout.
- **Employees & catalog UI:** `EmployeesContext` and `ProductsContext` split **`loadError`** (local IndexedDB / list read failed — can block login or POS catalog until retry) from **`syncError`** (background cloud sync failed — orange banner only; login and local catalog stay usable). Sync callbacks must never set the load channel. Products `loadData` awaits `initializeLocalDatabase()` before Dexie reads to avoid cold-start races.
- **Multi-till / multi-device:** Each scope keeps its own series/chain as configured; replication uses **stable IDs**, not shared editing of one invoice row.
- **Long-term evidence:** A **server copy** is valuable only if **immutable** after sync; the **client** can be tampered with in DevTools, so **DB rules** (triggers / strict RLS) and **locked RPCs** matter for the replica.

**Reversals — nota de crédito (NC) only for registered sales**

- A finalized sale with a **fiscal document / hash** must **not** be “undone” with **soft-delete** (`deleted_at`) or by deleting lines. The app already blocks local `deleteTransaction` when `fiscal_document_id` is set; **the same rule must hold on the server** (no `UPDATE` that hides or rewrites a sealed FT/FS).
- The **only** supported fiscal reversal is a **new** document: **nota de crédito (NC)** (see `runFiscalCreditNoteForTransaction` and the Transactions page). There is **no** in-app “void / anular document” path — that would pretend the original fiscal document did not exist; AT-compliant correction is always via **NC** (or the appropriate document type the law requires for the case).

**Production build — routes and tools that must not ship to end users**

- Today, routes such as **`/setup`** (DataSetup: populate / **clear** transaction data), **`/seed`** (SeedManagement), **`/receipt-demo`**, **`/design-system`**, and test pages (**`/cashier-testing`**, **`/electron-testing`**, **`/printer-test`**) are **not** environment-gated in `App.tsx` — they only use auth + permissions. For production POS deployments, **fail closed**:
  - **Recommended:** `VITE_ENABLE_DANGEROUS_DATA_TOOLS` (or `import.meta.env.DEV`) — register these routes **only** when the flag is on; set it **false** in production `.env` and verify in CI that production builds never enable it.
  - Use a **separate** Supabase project (or schema) for dev; never run mass-delete seed tools against a real tenant.
  - **Electron:** Production builds should not point at dev-only APIs or include dev “reset” entry points in the menu.

**Server / sync hardening (defense in depth)**

- Harden or restrict **`upsert_transaction_with_items`** and any sync path that **replaces** `transaction_items` so **sealed** rows cannot be rewritten by a generic upsert. Prefer **insert-once** for new docs and **append-only** semantics for fiscal evidence.

---

## Required Documentation

Before implementing features, reference:
- `STYLE_GUIDE.md` - UI/design patterns, colors, typography
- `DEVELOPMENT_GUIDE.md` - Architecture, TypeScript, components

---

## Git Conventions

### Never commit working artifacts

`artifacts/` and `.scratch/` are gitignored **on purpose** (screenshots, debug output, design references, temp notes). Do not commit them, do not remove them from `.gitignore`, and do not `git add -f` files inside them. If something in there turns out to be a real deliverable, move it to `docs/` first.

```
feat: add new feature
fix: resolve bug
docs: documentation changes
refactor: code restructuring
test: add/update tests
style: formatting, no code change
```

---
> Source: [portoyounes01/easysoft](https://github.com/portoyounes01/easysoft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
