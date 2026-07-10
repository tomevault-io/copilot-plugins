## project-structure

> This is a Next.js 15 template for Stacks blockchain applications with TypeScript, Tailwind CSS, and Biome for linting/formatting.


# Project Structure Guide

This is a Next.js 15 template for Stacks blockchain applications with TypeScript, Tailwind CSS, and Biome for linting/formatting.

## Key Directories

- **`app/`** - Next.js 15 App Router directory
  - [layout.tsx](mdc:app/layout.tsx) - Root layout with AuthSessionProvider
  - [page.tsx](mdc:app/page.tsx) - Home page component
  - [globals.css](mdc:app/globals.css) - Global styles with Tailwind CSS

- **`components/`** - Reusable UI components
  - `ui/` - Base UI components (Button, Card, Input) using Radix UI and CVA
  - [button.tsx](mdc:components/ui/button.tsx) - Button component with variants
  - [card.tsx](mdc:components/ui/card.tsx) - Card component
  - [input.tsx](mdc:components/ui/input.tsx) - Input component

- **`lib/`** - Utility libraries and configuration
  - `config/` - Environment configuration with Zod validation
  - [client.ts](mdc:lib/config/client.ts) - Client-side configuration
  - [server.ts](mdc:lib/config/server.ts) - Server-side configuration
  - [utils.ts](mdc:lib/utils.ts) - Utility functions

- **`hooks/`** - Custom React hooks
  - [wallet.ts](mdc:hooks/wallet.ts) - Stacks wallet connection hook

- **`providers/`** - React context providers
  - [auth-session-provider.tsx](mdc:providers/auth-session-provider.tsx) - Authentication session management


## Key Files

- [package.json](mdc:package.json) - Project dependencies and scripts
- [tsconfig.json](mdc:tsconfig.json) - TypeScript configuration
- [biome.json](mdc:biome.json) - Biome linting/formatting configuration
- [components.json](mdc:components.json) - UI components configuration
- [next.config.ts](mdc:next.config.ts) - Next.js configuration

## Architecture Patterns

- **App Router**: Uses Next.js 15 App Router for routing
- **TypeScript**: Full TypeScript support with strict configuration
- **Component Library**: Radix UI primitives with CVA for styling variants
- **State Management**: React Context for authentication state
- **Wallet Integration**: Stacks Connect for wallet functionality
- **Validation**: Zod schemas for environment and data validation

---
> Source: [Mcsavvy/stacks-next-template](https://github.com/Mcsavvy/stacks-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
