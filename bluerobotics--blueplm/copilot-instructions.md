## architecture

> Source folder structure and architecture overview


# BluePLM Architecture

## Source Folder Structure

```
src/
├── app/                    # App entry point
│   ├── App.tsx            # Main component, providers, layout
│   └── main.tsx           # React DOM entry
│
├── components/            # Reusable UI components
│   ├── core/             # Primitives: Dialog, Loader, Toast, ErrorBoundary
│   ├── effects/          # Visual effects (seasonal themes)
│   ├── layout/           # App shell: Sidebar, Topbar, Panels
│   └── shared/           # Cross-feature components
│
├── features/             # Feature modules (vertical slices)
│   ├── source/           # File browser, explorer, workflows
│   ├── settings/         # All settings screens
│   ├── search/           # Command palette, search views
│   ├── change-control/   # ECR, ECO, deviations
│   ├── supply-chain/     # Suppliers, RFQ, portal
│   ├── integrations/     # Google Drive, SolidWorks
│   ├── notifications/    # Notification center
│   ├── items/            # BOMs, products
│   └── dev-tools/        # Terminal, logs, performance
│
├── stores/               # Zustand state (see zustand.mdc)
│   ├── pdmStore.ts      # Combined store with persistence
│   ├── slices/          # Individual slice creators
│   ├── types.ts         # All state/action types
│   ├── selectors.ts     # Memoized derived state
│   └── migrations.ts    # Store version migrations
│
├── hooks/               # Shared React hooks
│   ├── useAuth.ts
│   ├── useClipboard.ts
│   ├── useKeyboardShortcuts.ts
│   └── ...
│
├── lib/                 # Core utilities & services
│   ├── supabase/       # Supabase client & queries
│   ├── commands/       # CLI command handlers
│   ├── fileOperations/ # File sync logic
│   ├── i18n/           # Translations
│   └── utils/          # Pure utility functions
│
├── types/              # Shared TypeScript types
│   ├── pdm.ts         # Core domain types
│   ├── modules.ts     # Module configuration
│   ├── workflow.ts    # Workflow types
│   └── supabase.ts    # Generated DB types
│
└── constants/          # App-wide constants
```

## Feature Module Structure

Each feature follows this pattern:

```
features/my-feature/
├── index.ts           # Public exports (barrel)
├── MyFeatureView.tsx  # Main view component
├── components/        # Feature-specific components
├── hooks/            # Feature-specific hooks
├── types.ts          # Feature types
└── utils.ts          # Feature utilities
```

## Import Aliases

Use `@/` alias for clean imports:

```typescript
import { usePDMStore } from '@/stores'
import { Button } from '@/components/shared'
import { supabase } from '@/lib/supabase'
import type { PDMFile } from '@/types/pdm'
```

## Component Placement Decision Tree

```
Is it a primitive (Dialog, Toast, Loader)?
  → src/components/core/

Is it app shell (Sidebar, Topbar)?
  → src/components/layout/

Is it used across multiple features?
  → src/components/shared/

Is it feature-specific?
  → src/features/{feature}/components/
```

## Adding a New Feature

1. Create folder: `src/features/my-feature/`
2. Add view: `MyFeatureView.tsx`
3. Export from `index.ts`
4. Add to sidebar (if needed): see `src/stores/types.ts` → `SidebarView`
5. Add slice (if state needed): see `zustand.mdc`

## Key Patterns

### Barrel Exports

Every folder has an `index.ts` that exports public API:

```typescript
// src/features/my-feature/index.ts
export { MyFeatureView } from './MyFeatureView'
export type { MyFeatureProps } from './types'
```

### Type Colocation

- **Shared types** → `src/types/`
- **Feature types** → `src/features/{feature}/types.ts`
- **Store types** → `src/stores/types.ts`

### Hook Organization

- **Shared hooks** → `src/hooks/`
- **Feature hooks** → `src/features/{feature}/hooks/`
- **Store selectors** → `src/stores/selectors.ts`

---
> Source: [bluerobotics/bluePLM](https://github.com/bluerobotics/bluePLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
