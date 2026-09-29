## synthcheckers

> npm run dev      # Dev server on port 5000

# SynthCheckers

## Quick Start
```bash
npm install
npm run dev      # Dev server on port 5000
npm run check    # TypeScript checking
npm test         # Run tests
npm run build    # Production build
```

## Documentation
- `ARCHITECTURE.md` - System design, data flow, components
- `CONTRIBUTING.md` - Coding standards and conventions

## Tech Stack
- **Frontend**: React 18 + TypeScript + Three.js + Vite 7
- **State**: Zustand (subscribeWithSelector middleware)
- **Backend**: Express 5 + Firebase (Auth, Firestore)
- **Testing**: Vitest + React Testing Library
- **Routing**: React Router 7

## Key Directories
```
client/src/
  components/     # React components (game/, auth/, ui/)
  lib/stores/     # Zustand stores (useCheckersStore, useAuthStore, useOnlineGameStore)
  lib/checkers/   # Game logic (rules.ts, ai.ts, types.ts)
  lib/elo/        # ELO rating calculations
  services/       # Firebase service classes
  types/          # TypeScript definitions
  __tests__/      # Tests
```

## Critical Patterns

### Zustand Store
```typescript
import { create } from "zustand";
import { subscribeWithSelector } from "zustand/middleware";

export const useStore = create<State>()(
  subscribeWithSelector((set, get) => ({
    // state and actions
  }))
);
```

### Firebase Service
```typescript
import { getFirebaseDb } from '../lib/firebase';

class Service {
  async method() {
    const db = await getFirebaseDb(); // Always async getter
    // use db...
  }
}
export const service = new Service();
```

## Do NOT Modify
- `client/src/components/ui/*.tsx` (shadcn primitives)
- `firestore.rules` (security review required)
- `.env` files

## Verification
1. `npm run check` - TypeScript passes
2. `npm test` - Tests run
3. `npm run dev` - Server starts (check console)

## Mandatory Checklist Before Committing

### After Dependency Changes
- **Always regenerate package-lock.json** after using `--legacy-peer-deps` or making multiple dependency changes
- Run `rm package-lock.json && npm install` to ensure lock file is in sync
- Verify with `npm ci` (what CI uses) - if it fails locally, it will fail in GitHub Actions
- Never commit a lock file that causes `npm ci` to fail with "Missing: X from lock file"

### After Architecture/Tech Stack Changes
- **Update CLAUDE.md** - Tech Stack section, patterns, any new conventions
- **Update ARCHITECTURE.md** - Dependencies section, component changes, new patterns
- Examples of changes requiring doc updates:
  - Major version upgrades (Express 4→5, Vite 5→7, etc.)
  - New libraries or removed dependencies
  - API changes (route syntax, component APIs)
  - New files that establish patterns (e.g., type declaration files)

---
> Source: [JakubAnderwald/SynthCheckers](https://github.com/JakubAnderwald/SynthCheckers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
