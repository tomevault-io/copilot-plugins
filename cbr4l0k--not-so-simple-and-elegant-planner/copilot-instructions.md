## not-so-simple-and-elegant-planner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- **Development**: `pnpm dev` - Start Vite dev server with HMR
- **Build**: `pnpm build` - TypeScript compilation (`tsc -b`) followed by Vite production build
- **Lint**: `pnpm lint` - Run ESLint
- **Preview**: `pnpm preview` - Preview production build locally

## Architecture

This is a strategic planning application built with **Hexagonal Architecture** (Ports and Adapters pattern). The architecture enforces strict separation of concerns:

### Layer Structure

```
src/
├── domain/           # Core business logic (framework-agnostic)
│   ├── entities/     # Domain models (Vision, BreakthroughObjective, AnnualGoal, Initiative, Task)
│   ├── ports/        # Interface contracts
│   │   ├── repositories/  # Data persistence interfaces
│   │   └── services/      # Domain service interfaces (CPMCalculator, PriorityMatrixCalculator)
│   └── services/     # Domain service implementations (CPMService, PriorityMatrixService, DateUtilities)
├── application/      # Application layer
│   ├── AppContext.tsx     # Dependency injection container
│   └── hooks/        # React hooks that orchestrate domain logic
├── infrastructure/   # External adapters
│   ├── repositories/ # InMemory implementations of repository ports
│   └── data/         # Initial data fixtures
└── ui/              # Presentation layer
    ├── App.tsx      # DI setup and app shell
    └── components/  # React UI components
```

### Dependency Rules

**Critical**: Dependencies flow inward only. Domain layer has ZERO dependencies on outer layers.

- **Domain** depends on: Nothing (pure TypeScript interfaces and classes)
- **Application** depends on: Domain ports (interfaces only)
- **Infrastructure** depends on: Domain ports (implements the interfaces)
- **UI** depends on: Application hooks and infrastructure implementations

### Dependency Injection

All dependencies are injected through `AppContext` (see `src/application/AppContext.tsx`):

1. `src/ui/App.tsx` instantiates concrete implementations (repositories, services)
2. These are passed to `AppProvider` as the `dependencies` prop
3. Hooks use `useAppContext()` to access injected dependencies
4. This allows swapping implementations without changing business logic

Example from `src/ui/App.tsx`:
```typescript
const visionRepository = new InMemoryVisionRepository(initialVision);
const cpmCalculator = new CPMService();

const dependencies = {
  visionRepository,
  cpmCalculator,
  // ... other dependencies
};

<AppProvider dependencies={dependencies}>
```

### Domain Concepts

The application implements strategic planning using **Hoshin Kanri** methodology:

- **Vision**: Long-term (multi-year) strategic vision
- **Breakthrough Objectives**: Major goals that support the vision
- **Annual Goals**: Yearly targets linked to breakthrough objectives
- **Initiatives**: Projects with tasks that accomplish annual goals
- **CPM (Critical Path Method)**: Calculates project schedules and critical paths
- **Priority Matrix**: Categorizes tasks by strategic importance and time criticality

## TypeScript Configuration

**IMPORTANT**: This project uses strict TypeScript settings:

- `verbatimModuleSyntax: true` in `tsconfig.app.json`
- `erasableSyntaxOnly: true` enabled

### Import Rules

You **MUST** follow these import patterns or the build will fail:

- **Type-only imports** (interfaces, types): Use `import type`
  ```typescript
  import type { Vision, AnnualGoal } from '../domain/entities';
  import type { VisionRepository } from '../domain/ports/repositories';
  ```

- **Value imports** (classes, functions, hooks): Use regular `import`
  ```typescript
  import { CPMService } from '../domain/services';
  import { InMemoryVisionRepository } from '../infrastructure/repositories';
  import { useVision } from '../application/hooks/useVision';
  ```

- **Mixed imports**: Separate into two statements
  ```typescript
  import type { VisionRepository } from '../../domain/ports/repositories';
  import type { Vision } from '../../domain/entities';
  // NOT: import { VisionRepository, Vision } from '...'
  ```

### Common Mistakes

❌ **Wrong**: Importing interfaces without `type` keyword
```typescript
import { Vision, AnnualGoal } from '../domain/entities';
```

✅ **Correct**:
```typescript
import type { Vision, AnnualGoal } from '../domain/entities';
```

❌ **Wrong**: Using `import type` for classes or functions
```typescript
import type { InMemoryVisionRepository } from '../infrastructure/repositories';
const repo = new InMemoryVisionRepository(data); // ERROR: cannot use as value
```

✅ **Correct**:
```typescript
import { InMemoryVisionRepository } from '../infrastructure/repositories';
const repo = new InMemoryVisionRepository(data);
```

## Repository Pattern

All repositories implement a common pattern:

1. Implement a port interface (e.g., `VisionRepository`)
2. Store data in private class property
3. Provide `subscribe()` method for reactive updates
4. Call `notifyListeners()` after mutations
5. Hooks subscribe to changes via `useEffect`

When checking if a repository is a specific implementation (e.g., for subscribing), import the class normally (not with `import type`):

```typescript
// In hooks:
import { InMemoryVisionRepository } from '../../infrastructure/repositories';

if (visionRepository instanceof InMemoryVisionRepository) {
  const unsubscribe = visionRepository.subscribe(() => {
    // handle update
  });
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cbr4l0k) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
