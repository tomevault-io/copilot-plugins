## evmint

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server with host access for external connections
- `npm run build` - Build the application for production
- `npm run preview` - Preview the production build locally

## Production Deployment

This app is production-ready and optimized for deployment to:
- **Firebase Hosting**: Fast CDN delivery with excellent caching

### Build Requirements
- Run `npm run build` to create optimized production build in `dist/` folder
- Ensure `VITE_PRIVY_APP_ID` environment variable is configured for target environment
- Test locally with `npm run preview` before deployment
- All unused dependencies and files have been removed for optimal bundle size

## Project Architecture

This is a React TypeScript application built with Vite that creates a token launcher interface for the Base blockchain. Key architectural components:

### Core Stack
- **Frontend**: React 18 + TypeScript + Vite
- **Styling**: Tailwind CSS with PostCSS
- **Animations**: Framer Motion for complex UI animations and transitions
- **Routing**: React Router DOM with route-based page structure

### Web3 Integration
- **Wallet Connection**: Privy for authentication and embedded wallets
- **Blockchain Interaction**: Wagmi v2 + Viem for Web3 operations
- **Network**: Configured for Base mainnet and Base Sepolia testnet
- **State Management**: TanStack React Query for async state

### Provider Architecture
The app uses a nested provider pattern:
```
PrivyProvider (auth) 
  → QueryClientProvider (data fetching)
    → WagmiProvider (Web3)
      → Router (navigation)
```

### Configuration
- Web3 configuration is centralized in `src/config/web3.ts`
- Privy app ID can be configured via `VITE_PRIVY_APP_ID` environment variable
- Supports both Base mainnet and testnet chains

### Page Structure
- `/` - HomePage: Landing page with hero section
- `/create` - CreateTokenPage: Token creation form with animated UI
- `/tokens` - TokensPage: Token management interface  
- `/liquidity` - LiquidityPage: Liquidity management
- `/guide` - GuidePage: User documentation

### Component Patterns
- Heavy use of Framer Motion variants for consistent animations
- Glassmorphism design with backdrop blur effects
- Responsive design with mobile-first approach
- Form validation and loading states with visual feedback

## Key Dependencies
- `@privy-io/react-auth` and `@privy-io/wagmi` for wallet integration
- `wagmi` v2 with `viem` for Ethereum interactions
- `framer-motion` for animations (used extensively throughout UI)
- `@tanstack/react-query` for server state management

## Uniswap V2 Integration
The app integrates with Uniswap V2 for liquidity management on Base network:

### Core Features
- **Add Liquidity**: Two-step process (approve → add) with comprehensive transaction monitoring
- **Remove Liquidity**: Two-step process (approve LP tokens → remove) with automatic balance detection
- **LP Token Management**: Automatic storage and retrieval of LP token addresses for easy withdrawal
- **Transaction Safety**: Proper slippage protection and minimum amount calculations

### Base Network Contracts
- **Base Mainnet**: Production deployment
  - UniswapV2Factory: `0x8909Dc15e40173Ff4699343b6eB8132c65e18eC6`
  - UniswapV2Router: `0x4752ba5dbc23f44d87826276bf6fd6b1c372ad24`
  - WETH: `0x4200000000000000000000000000000000000006`

- **Base Sepolia**: Not supported (Uniswap V2 not deployed on Base Sepolia)

### Implementation Guidelines
- All liquidity operations use wagmi's `useWriteContract` with async patterns
- Proper error handling with categorized error types (USER_REJECTED, GAS_ERROR, etc.)
- Transaction receipts are parsed to extract LP token addresses automatically
- LP tokens are stored in localStorage for easy access in withdrawal flows
- Minimum amounts calculated dynamically based on pool state to prevent MEV attacks

### Transaction Flow
1. **Add Liquidity**: Select tokens → Set amounts → Approve → Add → Success modal with DEXScreener link
2. **Remove Liquidity**: Select LP token → Set amount → Approve LP → Remove → Success modal with transaction link

## Development Notes
- Uses strict TypeScript configuration with separate app and node configs
- ESLint configured with React hooks and TypeScript rules
- Vite optimized for Web3 dependencies with specific exclusions for `@base-org/account`
- No test framework currently configured

## UI/UX Design Guidelines
Following established design principles for optimal user experience:

### Design System
- **Consistent Animations**: All components use Framer Motion variants from `src/styles/designSystem.ts`
- **Glass Morphism**: Primary design language with backdrop blur effects and subtle borders
- **Color Palette**: Blue/purple/cyan gradients with dark theme base (gray-900)
- **Typography**: Hierarchical system with gradient text for emphasis
- **Spacing**: 8px base unit with consistent padding/margin patterns

### Navigation
- **Desktop**: Horizontal navigation with icons and hover states
- **Mobile**: Slide-down menu with smooth animations and touch-friendly targets
- **Active State**: Visual indicators for current page with background highlights
- **Accessibility**: Proper ARIA labels and keyboard navigation support

### Component Patterns
- **Glass Cards**: Consistent container pattern with hover effects
- **Loading States**: Unified spinner and skeleton loading patterns
- **Form Controls**: Consistent input styling with focus states and validation
- **Buttons**: Primary (gradient) and secondary (outline) button patterns
- **Status Indicators**: Color-coded feedback for wallet, network, and transaction states

### User Flow Optimization
- **Progressive Disclosure**: Information revealed contextually based on user state
- **Clear CTAs**: Prominent call-to-action buttons with descriptive labels
- **Error Prevention**: Inline validation and clear error messages
- **Success Feedback**: Immediate confirmation of completed actions

## Firebase Analytics Integration

Comprehensive analytics tracking implemented for optimal user insights and debugging:

### Core Analytics Events
- **Page Views**: All pages track user navigation patterns
  - `page_view` event with `page_name` parameter
  - Tracks: home, create, tokens, liquidity, guides, faq, legal pages

- **Token Creation Events**:
  - `token_created` - Successful token deployment with metadata (name, symbol, supply, network)
  - `token_deployment_result` - Success/failure tracking with error details
  - Form submission tracking for UX optimization

- **Liquidity Management Events**:
  - `liquidity_added` - Track successful liquidity additions (token address, amounts, network)
  - `liquidity_removed` - Track liquidity withdrawals (LP token amount, network)
  - Transaction progress and completion tracking

- **User Interaction Events**:
  - `wallet_connected` / `wallet_disconnected` - Wallet connection patterns
  - `network_switched` - Network switching behavior
  - `button_clicked` - Key UI interaction tracking
  - `form_submitted` - Form completion rates

- **Error Tracking**:
  - `error_occurred` - Comprehensive error logging with type, message, location
  - User action context for debugging
  - Categorized error types (transaction, validation, network, etc.)

### Implementation Details
- **Firebase Configuration**: `src/config/firebase.ts` with conditional analytics loading
- **Analytics Provider**: `src/components/FirebaseProvider.tsx` with React context
- **Utility Functions**: `src/utils/analytics.ts` with typed event tracking functions
- **Hook Integration**: Analytics automatically triggered on user actions via useEffect hooks
- **Error Boundaries**: Graceful handling when analytics unavailable

### Analytics Dashboard Benefits
- **User Journey**: Complete funnel from landing to token creation
- **Error Monitoring**: Real-time error tracking and resolution insights  
- **Feature Usage**: Popular features and optimization opportunities
- **Performance Metrics**: Transaction success rates and completion times
- **User Retention**: Page engagement and return patterns

This 80/20 implementation provides maximum insight with minimal overhead, perfect for startup analytics needs.

---
> Source: [alexandreelkhoury/evmint](https://github.com/alexandreelkhoury/evmint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
