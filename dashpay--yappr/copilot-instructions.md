## yappr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linting
npm run lint
```

### Dash Platform Contract Deployment
```bash
# Register contract on Dash Platform
node register-contract.js

# Register contract with specific nonce
node register-contract-with-nonce.js
```

### Testing DPNS Resolution
```bash
node test-dpns-resolve.js
```

## Architecture Overview

Yappr is a decentralized social media platform built with Next.js 14 and Dash Platform. It follows a modern React architecture with server-side rendering capabilities.

### Key Architectural Decisions

1. **Next.js App Router**: Uses the new App Router pattern for improved performance and server components
2. **Decentralized Data Storage**: All user data is stored on Dash Platform using data contracts
3. **WebAssembly Integration**: Dash SDK runs in the browser via WASM for direct blockchain interaction
4. **Component-Based Architecture**: Modular React components with clear separation of concerns

### Core Systems

#### Authentication System
- Managed through `contexts/auth-context.tsx`
- Integrates with Dash Platform identities
- Handles wallet connection and identity management

#### SDK Integration
- `contexts/sdk-context.tsx` provides Dash SDK access throughout the app
- WebAssembly files in `/public/dash-wasm/` enable browser-based blockchain operations
- Supports both mainnet and testnet configurations

#### Data Contract Structure
The social platform uses 12 document types. The actual registered contract is in `contracts/yappr-social-contract-actual.json`:
- User profiles and avatars (separate documents for flexibility)
- Posts with 500 character limit and optional media (NO authorId field - uses $ownerId system field)
- Social interactions (likes, reposts, follows) - use $ownerId not userId
- Private features (bookmarks, lists, blocks, mutes)
- Direct messages and notifications

**IMPORTANT**: The post document does NOT have an `authorId` field. It uses the system field `$ownerId` which is automatically set by the platform. When creating posts, only include content-related fields.

#### DPNS Integration
- Usernames are managed through Dash Platform Name Service
- Profile documents don't store usernames directly
- Components in `components/dpns/` handle name resolution and display

### Important Patterns

1. **State Management**: Uses Zustand for global state (see `lib/store.ts`)
2. **Styling**: Tailwind CSS with custom Yappr design system defined in `tailwind.config.js`
3. **Component Organization**:
   - `components/ui/` - Reusable, styled UI primitives
   - `components/layout/` - Page layout components
   - `components/post/` - Post-specific components
   - `components/compose/` - Post creation UI

4. **Type Safety**: Strict TypeScript configuration with comprehensive types in `types/` directory
5. **Mock Data**: Development uses mock data from `lib/mock-data.ts` when not connected to Dash Platform

### Development Considerations

- Always check WebAssembly compatibility when modifying SDK-related code
- Respect the 500 character limit for posts in UI and validation
- Avatar data is encoded as base64 strings with version control
- Use existing UI components from Radix UI before creating custom ones
- Follow the established color palette and design system in Tailwind config
- Ensure all blockchain operations handle loading and error states properly

### Known Issues

#### DAPI Gateway Timeouts
The `wait_for_state_transition_result` function frequently times out with 504 Gateway Timeout errors even when transactions succeed. This is a known issue with DAPI nodes. The application handles this gracefully by:
- Attempting to wait for confirmation with a short timeout
- Assuming success if the broadcast was successful but wait times out
- Not blocking the UI on confirmation (posts appear immediately after broadcast)

---
> Source: [dashpay/yappr](https://github.com/dashpay/yappr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
