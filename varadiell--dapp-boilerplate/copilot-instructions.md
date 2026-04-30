## dapp-boilerplate

> Next.js App Router, components, styling, and frontend conventions.


# Frontend Development Guidelines

## Next.js App Router

Follow the structure in [frontend/src/app/](mdc:frontend/src/app/):

- Use the app router with `page.tsx` files in each route directory
- Implement layouts with [frontend/src/app/layout.tsx](mdc:frontend/src/app/layout.tsx)
- Use `'use client'` where the tree needs browser-only APIs (Web3, i18n provider, etc.)

## Component Architecture

### UI Components

Use shadcn/ui components from [frontend/src/components/ui/](mdc:frontend/src/components/ui/) for consistent design:

- Import from `@/components/ui/` for reusable components
- Follow the established component patterns with proper TypeScript interfaces
- Use `cn()` utility from [frontend/src/lib/utils.ts](mdc:frontend/src/lib/utils.ts) for conditional classes

### Shared Components

DApp-specific components in [frontend/src/components/shared/](mdc:frontend/src/components/shared/):

- Follow the naming convention: `component-name.tsx`
- Use proper TypeScript interfaces for props
- Implement proper error handling and loading states

## Web3 Integration

### Contract Integration

- Use the contract configuration from [frontend/src/contracts/ballot.contract.ts](mdc:frontend/src/contracts/ballot.contract.ts)
- Follow [frontend/src/hooks/useContract.ts](mdc:frontend/src/hooks/useContract.ts) for writes and receipts; use hooks under [frontend/src/hooks/ballot/](mdc:frontend/src/hooks/ballot/) for Ballot reads and event logs
- Always handle connection states and transaction statuses

### Wagmi Configuration

- Configure `WagmiAdapter`, networks, and transports in [frontend/src/lib/wagmi.config.ts](mdc:frontend/src/lib/wagmi.config.ts) (networks from `@reown/appkit/networks`)
- Use environment variables for API keys and endpoints
- Supported networks: Hardhat (local), Sepolia, Base Sepolia, Ethereum mainnet, Base

### Internationalization

- i18n is initialized in [frontend/src/lib/i18n/client.ts](mdc:frontend/src/lib/i18n/client.ts); wrap the app with [frontend/src/contexts/i18n-provider.tsx](mdc:frontend/src/contexts/i18n-provider.tsx)
- Add copy in [frontend/src/locales/](mdc:frontend/src/locales/) (`en/common.json`, `fr/common.json`); use `useTranslation` from `react-i18next` in components

### Client state (Zustand)

- Use [frontend/src/stores/use-data-store.ts](mdc:frontend/src/stores/use-data-store.ts) for derived ballot UI state; keep Wagmi/React Query as the source of truth for chain data and sync via [frontend/src/contexts/ballot-data-store-sync.tsx](mdc:frontend/src/contexts/ballot-data-store-sync.tsx)

### Provider Setup

The Web3Provider in [frontend/src/contexts/web3-provider.tsx](mdc:frontend/src/contexts/web3-provider.tsx) wraps:

- WagmiProvider for blockchain interactions
- QueryClientProvider for React Query
- Reown for wallet connections

## Styling

- Use Tailwind CSS with the configuration in [frontend/tailwind.config.ts](mdc:frontend/tailwind.config.ts)
- Follow the design system with CSS variables for theming
- Use the theme provider from [frontend/src/contexts/theme-provider.tsx](mdc:frontend/src/contexts/theme-provider.tsx)

## Development Commands

From [frontend/package.json](mdc:frontend/package.json):

- `bun run dev` - Start development server
- `bun run build` - Build for production
- `bun run lint` - Prettier check on JS/TS/TSX
- `bun run cspell` - Spell checking

## Best Practices

1. **TypeScript**: Use strict TypeScript with proper interfaces
2. **Error Handling**: Implement proper error boundaries and user feedback
3. **Loading States**: Show loading indicators for async operations
4. **Responsive Design**: Ensure components work on all screen sizes
5. **Accessibility**: Follow WCAG guidelines and use semantic HTML

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Varadiell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
