## staywise

> StayWise core project standards and architecture


# StayWise Project Standards

## Tech Stack
- Expo SDK 54 + expo-router (file-based routing)
- TypeScript strict mode
- Supabase (Postgres + Auth + RLS)
- TanStack Query v5 for server state
- react-hook-form + Zod for forms
- NativeWind for styling

## Data Flow
```
Component → React Query Hook → API Function → Supabase → Cache Update
```

## File Placement
| What | Where |
|------|-------|
| Screen | `app/(group)/screen-name.tsx` |
| Component | `src/components/ui/` or `src/components/shared/` |
| API function | `src/services/api/` |
| Query hook | `src/services/queries/` |
| Schema | `src/schemas/` |

## Commands
```bash
pnpm install          # Install deps
pnpm start            # Dev server
npx tsc --noEmit      # Type check
npx expo start --clear # Clear cache
```

## Never Do
- Use `any` type (use `unknown` instead)
- Skip Zod validation on user inputs
- Create inline styles (use NativeWind)
- Modify database without migration
- Store secrets in code
- Use emojis in code, comments, or commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sangameshbiradar777) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
