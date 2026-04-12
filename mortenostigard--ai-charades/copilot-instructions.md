## ai-charades

> React component patterns and standards for AI Charades

# React Component Standards

## Component Structure
Always use the following structure for new components.

```tsx
// 1. Imports in the correct order (see below)
import { useCallback } from 'react';
import { useGameState } from '@/hooks/useGameState';
import { LoadingSpinner } from '@/components/ui/loading-spinner';
import { ErrorMessage } from '@/components/ui/error-message';

// 2. Props interface (if props are accepted)
interface ComponentProps {
  // Required props
  readonly required: string;
  // Optional props with defaults
  readonly optional?: boolean;
}

// 3. Component definition
export function Component({ required, optional = false }: ComponentProps) {
  // 4. Hooks at the top
  const { loading, error } = useGameState();

  // 5. Event handlers memoized with useCallback to prevent unnecessary re-renders
  const handleAction = useCallback(() => {
    // Event logic
  }, [dependencies]);

  // 6. Early returns for loading/error states
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  // 7. Main render
  return (
    <div className="mobile-friendly-class">
      {/* Content */}
    </div>
  );
}
```

## Core Requirements
- **Props Definition**: Components accepting props **MUST** have a TypeScript `interface`. Components that take no props do not need one.
- **Read-only Props**: All properties within a props `interface` **MUST** be marked as `readonly`.
- **Mobile-First**: All components must be designed for mobile, with minimum 44px touch targets.
- **State Handling**: Components with async operations **MUST** handle and display loading and error states.
- **Dependencies**: Hooks like `useEffect` and `useCallback` **MUST** have a complete and correct dependency array.
- **Zustand Optimization**: Use specific property selectors instead of entire state objects in dependencies to prevent unnecessary re-renders. Example: `const roomCode = useGameStore(state => state.gameState?.room.code)` instead of depending on the full `gameState`.
- **Styling**: Use `shadcn/ui` components and `tailwind-css` classes as the default.

## Game-Specific Component Categories
- **Views**: `ActorView`, `DirectorView`, `AudienceView` (role-specific UIs)
- **Shared**: `GameTimer`, `ScoreBoard`, `PromptDisplay` (common elements)
- **Layout**: `GameRoom`, `WaitingRoom` (containers)
- **UI**: `PlayerNameInput`, `SabotageButton` (interactive elements)

## Iconography
- **Primary Source**: Use icons from the `lucide-react` library whenever possible.
- **Consistency**: Avoid custom SVGs for icons that have a suitable equivalent in the library to maintain visual consistency.
- **Animation**: Use `tailwind-css` classes (e.g., `animate-spin`) to animate icons.

## Import Order
1.  React & Next.js imports (`react`, `next/router`, etc.)
2.  Third-party library imports (`framer-motion`, `socket.io-client`, `zustand`)
3.  Custom hook imports (`@/hooks/...`)
4.  Component imports (`@/components/ui/...`, `@/components/game/...`)
5.  Utility & Lib imports (`@/lib/...`)
6.  Type imports (`@/types`)
7.  Local styles or assets (if any)

@docs/tech_spec.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mortenostigard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mortenostigard)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
