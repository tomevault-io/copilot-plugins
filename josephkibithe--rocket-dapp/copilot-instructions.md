## rocket-dapp

> - `/pages` - Next.js page components

# Code Structure for ROCKET Web3 Prediction Market

## Directory Organization

### `/src` - Application Source Code
- `/pages` - Next.js page components
- `/components` - Reusable UI components
- `/styles` - Global styles and theme configuration
- `/hooks` - Custom React hooks
- `/utils` - Utility functions
- `/types` - TypeScript type definitions
- `/constants` - Application constants

### `/lib` - Core Libraries
- `lib/supabase.js` - Supabase client configuration
- `lib/wagmi.js` - Web3 configuration with wagmi/viem
- `lib/store.js` - Zustand global state management
- `lib/ipfs.js` - IPFS client for asset storage

### `/contracts` - Smart Contract Code
- `PredictionMarket.sol` - Main prediction market contract
- `interfaces/` - Solidity interfaces
- `test/` - Contract test files

### `/public` - Static Assets
- `/fonts` - Custom typography
- `/images` - Static image assets
- `/icons` - UI icons

### `/scripts` - Utility Scripts
- Contract deployment scripts
- Data migration utilities
- Build and optimization scripts

## Key Files

- `next.config.js` - Next.js configuration
- `tailwind.config.js` - Tailwind CSS customization
- `hardhat.config.js` - Hardhat blockchain configuration
- `package.json` - Project dependencies
- `tsconfig.json` - TypeScript configuration

## Code Organization Principles

1. **Component-first approach**: All UI elements are built as reusable components
2. **Server-side rendering** for public pages to optimize SEO and load time
3. **Client-side rendering** for authenticated routes with real-time updates
4. **Smart contract separation**: Contract logic is isolated in `/contracts` directory
5. **Custom hooks** for reusable stateful logic
6. **Type safety** across the entire codebase with TypeScript

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JosephKibithe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
