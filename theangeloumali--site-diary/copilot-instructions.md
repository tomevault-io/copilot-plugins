## site-diary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Yarn v1 monorepo containing a mobile app (React Native/Expo) and a web app (Next.js) that share a GraphQL API for site diary management. The web app serves as the GraphQL backend using Grats for schema generation, while the mobile app consumes the API.

## Prerequisites

- Node.js 22
- Yarn v1 (v1.22.22)
- Cocoapods (for iOS development)

## Monorepo Structure

```
apps/
  mobile/       # React Native/Expo mobile app with GraphQL client
  web/          # Next.js app with GraphQL API server (Grats-based)
packages/
  eslint-config/ # Shared ESLint configuration
```

## Development Commands

### Initial Setup

```bash
yarn install
```

### Mobile App (@untitled/mobile)

```bash
# First-time setup (iOS/Android native builds)
yarn workspace @untitled/mobile prebuild
yarn workspace @untitled/mobile ios
yarn workspace @untitled/mobile android

# Start dev server (if app already installed on simulator)
yarn dev:mobile
# Or: yarn workspace @untitled/mobile dev

# Generate GraphQL types from web API
yarn workspace @untitled/mobile codegen

# Type checking
yarn workspace @untitled/mobile typecheck

# Linting
yarn workspace @untitled/mobile lint
```

### Web App (@untitled/web)

```bash
# Start GraphQL API server (default: http://localhost:3000)
yarn dev:web
# Or: yarn workspace @untitled/web dev

# GraphQL endpoint: http://localhost:3000/api/graphql

# Build for production
yarn workspace @untitled/web build

# Start production server
yarn workspace @untitled/web start

# Regenerate GraphQL schema (from Grats decorators)
yarn workspace @untitled/web grats

# Type checking
yarn workspace @untitled/web typecheck

# Linting
yarn workspace @untitled/web lint
```

### Monorepo-wide Commands

```bash
# Run all CI checks (format, lint, typecheck)
yarn ci

# Format all code
yarn format

# Check formatting
yarn format-check

# Run linting across all workspaces
yarn lint

# Run type checking across all workspaces
yarn typecheck

# Check dependency consistency
yarn sherif-check

# Fix dependency inconsistencies
yarn sherif
```

## Architecture

### Web App (GraphQL API Backend)

- **Framework**: Next.js 15 with App Router and Turbopack
- **GraphQL**: Uses [Grats](https://grats.capt.dev) for code-first GraphQL schema generation
  - Schema is auto-generated in `schema.ts` by Grats CLI
  - Resolvers are defined in `src/app/api/graphql/query.ts` and `mutation.ts` using `@gql*` JSDoc decorators
  - GraphQL endpoint served via GraphQL Yoga at `/api/graphql`
- **Data Layer**: Currently uses static JSON data in `src/app/data/site-diary.json`
- **API Structure**:
  - GraphQL API: `/api/graphql` (query and mutation operations)
  - REST endpoints: `/api/site-diary` for site diary CRUD operations
- **Key Types**: Defined in `src/app/data/types.ts` (SiteDiary, Weather)

### Mobile App (GraphQL Client)

- **Framework**: Expo 54 with Expo Router (file-based routing)
- **GraphQL Client**: Apollo Client v4
- **Navigation**: Expo Router with tab-based navigation in `src/app/(tabs)`
- **Styling**: NativeWind (Tailwind CSS for React Native) with class-variance-authority
- **Code Generation**: GraphQL Codegen generates TypeScript types from web API schema
  - Config: `codegen.ts`
  - Generated types: `src/types/__generated__/graphql.ts`
  - Run codegen when GraphQL schema changes
- **Compiler**: React Compiler enabled for automatic optimization
- **Animation**: Reanimated with worklets support

### GraphQL Schema Workflow

1. Web app: Define/modify GraphQL types and resolvers in `src/app/api/graphql/` using Grats JSDoc decorators
2. Web app: Run `yarn workspace @untitled/web grats` to regenerate `schema.ts`
3. Ensure web dev server is running (`yarn dev:web`)
4. Mobile app: Run `yarn workspace @untitled/mobile codegen` to generate TypeScript types from the schema

### Development Workflow

1. Start web server first (mobile app codegen depends on it): `yarn dev:web`
2. For mobile development after first prebuild: `yarn dev:mobile`
3. When making GraphQL schema changes:
   - Update query/mutation files in web app with Grats decorators
   - Run `grats` to regenerate schema
   - Run mobile codegen to update types

## Technology Stack

### Shared

- TypeScript 5.9
- GraphQL 16.11
- React 19.1 (resolution override)
- Turbo for monorepo task orchestration
- Prettier with multiple plugins for code formatting

### Mobile-Specific

- Expo SDK 54
- Expo Router 6 (file-based routing)
- Apollo Client 4
- NativeWind (Tailwind for React Native)
- React Native Reanimated
- React Compiler (babel-plugin-react-compiler)

### Web-Specific

- Next.js 15 with App Router
- GraphQL Yoga for server
- Grats for schema generation

## Important Notes

- The mobile app GraphQL codegen expects the web server to be running at `http://localhost:3000/api/graphql`
- Both apps use React 19.1 via yarn resolutions
- Mobile app uses Expo Router with tabs layout - routes are in `src/app/(tabs)`
- Schema changes must be made through Grats decorators in the web app, not by manually editing `schema.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theangeloumali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
