## zksync-nuxt-starter

> Use npm for this project.

# CLAUDE.md

## Development Commands

Use npm for this project.

Use `npm run dev` to start the development server.

## Project Overview

This is a ZKsync Nuxt starter kit for building Web3 applications with wallet connectivity and blockchain interactions.
The project uses Nuxt 4 with Vue 3 and integrates ZKsync-specific features including SSO authentication.

## Architecture

### Core Stack

- **Framework**: Nuxt 4 with Vue 3.5
- **Web3**: Wagmi/Viem for blockchain interactions
- **Wallet**: Reown AppKit (formerly WalletConnect) for wallet connectivity
- **State Management**: Pinia stores
- **Styling**: Tailwind CSS v4 with DaisyUI components
- **UI Components**: Reka UI for headless components
- **Data Fetching**: TanStack Query (Vue Query)
- **Icons**: Fluent UI System Icons via @nuxt/icon

### Tools

- Use viem, not ethers library. (https://viem.sh/docs/getting-started)
- When using zksync-js, use viem. (https://matter-labs.github.io/zksync-js/latest/overview/index.html)
- Use wagmi for wallet management (https://wagmi.sh/vue/getting-started)
- Use wagmi/core for contracts (https://wagmi.sh/core/getting-started)
- Use tanstack query for fetching (https://tanstack.com/query/latest/docs/framework/vue/overview)

### Auto-imports

Nuxt auto-imports from:

- `app/composables/**` (including nested directories)
- `constants/` directory
- Wagmi Vue composables (pre-configured in `nuxt.config.ts`)
- Reown AppKit composables (pre-configured in `nuxt.config.ts`)

### General Instructions

- Use DaisyUI first, then reka-ui for more complex UI components. Don't worry about trying to match styling, I will update styling.
- Write out a plan and ask any questions to clarify the implementation before implementing
- Do not check types or eslint
- Use `~~` to reference the project root for imports, `~` for `./app`
- Don't worry about formatting/linting, the stop hook will format all changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsacoyote)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itsacoyote)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
