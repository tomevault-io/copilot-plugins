## hyperevm-tx-simulator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **HyperEVM Transaction Simulator** - a React-based web application for simulating blockchain transactions on the HyperEVM network. The app provides three simulation modes: single transactions, transaction bundles, and advanced state override scenarios.

## Development Commands

```bash
# Development server with hot reload
npm run dev

# Production build (TypeScript compilation + Vite bundling)
npm run build

# Code linting
npm run lint

# Preview production build
npm run preview
```

## Architecture

### Core Stack
- **Frontend**: React 19 + TypeScript + Vite
- **Blockchain**: Viem 2.33 + Wagmi 2.16 for Ethereum/HyperEVM interaction
- **Styling**: TailwindCSS 4.1 with utility-first approach
- **State**: React Query for async blockchain operations
- **Build**: Vite 7.0 with modern ESM setup

### Application Structure
```
src/
├── components/           # Three main simulator components
│   ├── TransactionSimulator.tsx     # Single transaction simulation
│   ├── BundleSimulator.tsx          # Multiple transaction bundles
│   └── StateOverrideSimulator.tsx   # Advanced state manipulation
├── config/
│   └── wagmi.ts         # HyperEVM chain configuration (chain ID 998)
└── App.tsx              # Tab-based navigation root
```

### Component Architecture
- **App.tsx**: Root component with tab navigation and React Query provider
- **TransactionSimulator**: Uses Wagmi's `simulateContract` for single function calls
- **BundleSimulator**: Uses Viem's `simulateCalls` for transaction sequences  
- **StateOverrideSimulator**: Advanced simulation with state manipulation (UI implemented, backend integration pending)

## HyperEVM Configuration

The HyperEVM chain is configured in `src/config/wagmi.ts`:
- Chain ID: 998
- Default RPC: http://localhost:8545
- Update the RPC URL for actual HyperEVM network deployment

## Key Implementation Notes

### Blockchain Integration
- Uses Viem for low-level blockchain operations
- Wagmi provides React hooks and higher-level abstractions
- All contract interactions require ABI parsing with `parseAbi()`
- Transaction simulation supports custom gas limits, block numbers, and sender addresses

### State Management
- React Query handles async blockchain operations and caching
- Component state manages form inputs and UI state
- No global state management - each simulator is self-contained

### Error Handling
- Try/catch blocks around all blockchain operations
- User-friendly error messages displayed in results panels
- TypeScript ensures type safety for blockchain parameters

## Documentation

Comprehensive Japanese documentation exists in `/docs/`:
- `product.md`: Detailed product requirements and feature specifications
- `impl.md`: Technical implementation guide with Viem/Wagmi usage patterns

## Current Limitations

- State override functionality shows UI but needs backend integration
- HyperEVM-specific precompiles not yet implemented
- No execution tracing or advanced analytics
- Missing test infrastructure

## Development Notes

- The build process is two-step: TypeScript compilation followed by Vite bundling
- TailwindCSS requires the `@tailwindcss/postcss` plugin in PostCSS config
- All blockchain addresses must be typed as `Address` from Viem
- React 19 is used with modern ESM modules throughout

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aki-0517)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aki-0517)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
