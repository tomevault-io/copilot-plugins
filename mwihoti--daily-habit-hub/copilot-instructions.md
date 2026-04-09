## daily-habit-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start Next.js dev server

# Build & Lint
npm run build        # Next.js production build (uses --webpack flag for Web3 compat)
npm run lint         # ESLint via next lint

# Testing
npm test             # Vitest in watch mode
npm run test:run     # Vitest single run (CI)

# Smart Contracts
npx hardhat compile                          # Compile Solidity contracts
npx hardhat run scripts/deploy.ts --network fuji   # Deploy to Avalanche Fuji testnet
npx hardhat verify --network fuji <address>       # Verify on Snowtrace

# Database
npx supabase migration new <name>   # Create new migration
npx supabase db push                # Apply migrations
```

**Run a single test file:**
```bash
npm run test:run -- __tests__/trainer-setup.test.tsx
```

## Architecture Overview

### Stack

- **Framework**: Next.js 15/16 App Router, React 19
- **Database/Auth**: Supabase (PostgreSQL + Auth via `@supabase/ssr`)
- **Styling**: Tailwind CSS + shadcn/ui (Radix UI primitives in `src/components/ui/`)
- **Web3**: Wagmi + RainbowKit + viem targeting Avalanche (Fuji testnet + mainnet)
- **Forms**: React Hook Form + Zod
- **Server state**: TanStack Query (React Query)
- **Testing**: Vitest + Testing Library, jsdom environment

### Directory Layout

```
app/                    # Next.js App Router — all pages and API routes
src/
  components/           # Shared UI (Layout, Navigation, StreakComponents, ui/)
  hooks/                # Custom React hooks
  lib/
    supabase/           # Supabase client factory (createClient)
    web3/               # Wagmi config, contract ABIs/addresses, artifacts/
contracts/              # Solidity smart contracts (HabitRegistry, HabitToken, AchievementNFT)
supabase/migrations/    # SQL migrations
__tests__/              # Vitest test files (currently trainer-setup.test.tsx)
typechain-types/        # Auto-generated contract TypeScript types
```

### Path Alias

`@/*` maps to `./src/*` (defined in `tsconfig.json`).

### Web3 / Smart Contracts

Three contracts deployed on Avalanche Fuji testnet (addresses in `.env`):

- **HabitRegistry** — records on-chain check-ins, rate-limited to 1/wallet/UTC day, mints 10 `$HABIT` tokens per check-in, triggers NFT milestones at 7/30/100 records
- **HabitToken** — ERC20 `$HABIT` capped-supply token, minted by HabitRegistry
- **AchievementNFT** — ERC721 (symbol: FITA), minted at streak milestones

The API route `/api/web3/record-habit` uses a **server-side admin wallet** (`PRIVATE_ADMIN_KEY`) to submit transactions on behalf of users.

Web3 libraries (wagmi, RainbowKit, viem) require special webpack transpilation config in `next.config.ts` plus Node module polyfill stubs.

**Known type issue**: `useReadContracts` causes TypeScript instantiation depth errors on Vercel — use raw `eth_call` via viem instead (see commit `5c38097`).

### API Routes

```
/api/auth/{signup,signin,user}
/api/trainers/profile           # CRUD for trainer profiles (auth required)
/api/trainers/route             # GET all trainers
/api/trainers/[id]/route        # GET trainer by ID
/api/web3/record-habit          # POST — admin wallet records habit on-chain
/api/conversations/route        # GET conversations
/api/conversations/[id]/messages/route
/api/ipfs/pin/route             # POST — pin to IPFS via Pinata
```

All protected routes check auth with `supabase.auth.getUser()` and return 401 if missing.

### Key Patterns

- **Supabase client**: Use `createClient()` from `@/lib/supabase/...` (SSR-compatible). Never use the raw `supabase-js` singleton directly in App Router pages.
- **Data fetching**: React Query for all server state; query keys follow `['resource', id]` convention.
- **Multi-step forms**: Wizard pattern used in `/trainer-setup` (5 steps). See `app/trainer-setup/page.tsx` as the reference implementation.
- **Toast notifications**: Use `sonner` (`toast.success/error`).
- **Icons**: Lucide React only.

### Environment Variables

See `.env.example` for all required variables:
- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `NEXT_PUBLIC_HABIT_REGISTRY_ADDRESS` / `_TOKEN_ADDRESS` / `_ACHIEVEMENT_NFT_ADDRESS`
- `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID`
- `PRIVATE_ADMIN_KEY` — **server-only**, never expose to client

### Testing Setup

Tests live in `__tests__/`. Coverage is scoped to `app/trainer-setup/**` and `app/api/trainers/**` in `vitest.config.ts`.

`vitest.setup.ts` provides global mocks for:
- `window.matchMedia`
- Next.js navigation hooks (`useRouter`, `useParams`, etc.)
- Supabase client + auth
- Wagmi (`useAccount`) and RainbowKit (`ConnectButton`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mwihoti)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mwihoti)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
