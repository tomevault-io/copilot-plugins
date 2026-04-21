## solana-demo-01

> - `app/` - Next.js application routes and pages

# Frontend Structure

## Directory Organization

### `src/`
- `app/` - Next.js application routes and pages
  - [page.tsx](mdc:src/app/page.tsx) - Main application page
  - [layout.tsx](mdc:src/app/layout.tsx) - Root layout component
- `components/` - Reusable React components
  - [stake/StakeManager.tsx](mdc:src/components/stake/StakeManager.tsx) - Staking interface
  - [bnpl/TransactionForm.tsx](mdc:src/components/bnpl/TransactionForm.tsx) - BNPL transaction form
- `hooks/` - Custom React hooks
  - [use-stake-pool.ts](mdc:src/hooks/use-stake-pool.ts) - Stake pool integration
- `lib/` - Utility functions and shared code
  - [constants.ts](mdc:src/lib/constants.ts) - Application constants
  - [utils.ts](mdc:src/lib/utils.ts) - Utility functions

## Key Components

### Staking Components
1. `StakeManager` - Main staking interface
   - Handles stake operations
   - Displays stake information
   - Manages rewards

2. `StakeForm` - Staking form component
   - Input validation
   - Amount processing
   - Error handling

### BNPL Components
1. `TransactionForm` - BNPL transaction processing
   - Card validation
   - Amount verification
   - Transaction submission

2. `TransactionHistory` - Transaction display
   - List transactions
   - Show status
   - Display details

## Development Guidelines
1. Use TypeScript for all components
2. Implement proper error handling
3. Follow React best practices
4. Maintain component documentation
5. Keep components modular and reusable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoasung01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
