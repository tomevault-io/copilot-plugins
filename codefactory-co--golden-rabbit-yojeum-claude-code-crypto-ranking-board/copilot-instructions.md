## golden-rabbit-yojeum-claude-code-crypto-ranking-board

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a cryptocurrency ranking dashboard built with Next.js 15.2.4 and React 19. It displays real-time information about the top cryptocurrencies including price, market cap, volume, and 24h change.

## Development Commands

```bash
# Install dependencies (using pnpm)
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run linting
pnpm lint
```

## Architecture & Key Components

### Core Structure
- **Framework**: Next.js App Router (app directory structure)
- **Main Component**: `crypto-ranking-board.tsx` - The primary component that renders the cryptocurrency ranking table
- **Entry Point**: `app/page.tsx` - Simple wrapper that imports and renders the crypto ranking board

### Data Flow
- Currently uses mock data hardcoded in `crypto-ranking-board.tsx`
- Planned integration with CoinAPI for real-time data (mentioned in code comments)
- Data structure defined by `CoinData` interface:
  ```typescript
  interface CoinData {
    rank: number
    name: string
    symbol: string
    price: number
    change24h: number
    marketCap: number
    volume: number
    logo: string
  }
  ```

### UI Architecture
- **Component Library**: shadcn/ui (Radix UI primitives + Tailwind CSS)
- **Styling**: Tailwind CSS with custom theme configuration
- **Theme**: Dark mode by default with CSS variables for theming
- **Icons**: Lucide React icons
- **Responsive Design**: Mobile-first approach with hidden columns on smaller screens

### Key Technical Decisions
- **TypeScript**: Strict mode enabled
- **Path Aliases**: `@/` prefix for imports (configured in tsconfig.json)
- **Build Configuration**: Currently has ESLint and TypeScript errors ignored during builds (see next.config.mjs)
- **Image Handling**: Unoptimized images enabled in Next.js config

## Important Notes

1. **No Testing Framework**: Currently no test setup. When adding tests, you'll need to configure a testing framework.

2. **Build Warnings Suppressed**: Both TypeScript and ESLint errors are ignored during builds. Consider enabling these checks:
   ```javascript
   // next.config.mjs
   eslint: { ignoreDuringBuilds: false }
   typescript: { ignoreBuildErrors: false }
   ```

3. **Mock Data**: The app currently displays static mock data for 8 cryptocurrencies. Real-time data integration is pending.

4. **UI Components**: Extensive UI component library available in `components/ui/`. These are shadcn/ui components that can be customized.

## Common Tasks

### Adding a New Cryptocurrency
1. Add the coin logo to `public/coin/[coinname].png`
2. Update the `coinData` array in `crypto-ranking-board.tsx`

### Modifying the Table Display
- Table structure is in `crypto-ranking-board.tsx`
- Responsive behavior controlled by `hidden md:table-cell` classes
- Number formatting handled by `formatNumber` and `formatPrice` utility functions

### Styling Changes
- Global styles: `app/globals.css`
- Theme configuration: `tailwind.config.ts`
- Component-specific styles use Tailwind utility classes

---
> Source: [codefactory-co/golden-rabbit-yojeum-claude-code-crypto-ranking-board](https://github.com/codefactory-co/golden-rabbit-yojeum-claude-code-crypto-ranking-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
