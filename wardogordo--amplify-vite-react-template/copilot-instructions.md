## amplify-vite-react-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md
    
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

**Requirements**: Node.js >= 20.20.0, npm >= 10.8.0

```bash
npm run dev        # Start Vite dev server
npm run build      # TypeScript compile + Vite production build
npm run lint       # ESLint with TypeScript rules (zero warnings allowed)
npm run preview    # Preview production build locally
```

**Amplify Backend Commands** (requires AWS credentials):
```bash
npx ampx sandbox   # Start local cloud sandbox for development
npx ampx generate outputs --app-id <app-id> --branch main  # Regenerate amplify_outputs.json
```

## Architecture Overview

This is an **AWS Amplify Gen 2** application using React, TypeScript, and Vite.

### Backend (`amplify/`)

Infrastructure-as-code using TypeScript:
- **`backend.ts`** - Orchestrates backend by combining auth and data resources
- **`auth/resource.ts`** - Cognito configuration (email-based login)
- **`data/resource.ts`** - GraphQL schema with DynamoDB, defines data models and authorization rules

The schema uses **owner-based authorization** (`allow.owner()`) for automatic per-user data isolation - no manual auth checks needed in application code. Default authorization mode is `userPool` (Cognito), so new models should use `allow.owner()` or `allow.authenticated()` for user-scoped access.

### Frontend (`src/`)

- **`main.tsx`** - Entry point; initializes Amplify SDK and wraps app in `<Authenticator>` for automatic auth UI
- **`App.tsx`** - Main component using `generateClient<Schema>()` for type-safe GraphQL operations

### Key Patterns

**Type-safe data client**: Import `Schema` from backend, use `generateClient<Schema>()` for compile-time GraphQL validation.

**Real-time subscriptions**: Use `client.models.ModelName.observeQuery().subscribe()` for live data updates instead of one-time fetches.

**Authentication**: The `<Authenticator>` wrapper handles login UI; access user/signOut via `useAuthenticator()` hook.

### Generated Files

- **`amplify_outputs.json`** - Runtime config generated from deployed backend (Cognito IDs, AppSync endpoint, etc.). Do not edit manually.

## Tech Stack

- React 18 with functional components and hooks
- TypeScript (strict)
- Vite 7
- AWS Amplify Gen 2 (Cognito, AppSync, DynamoDB)
- @aws-amplify/ui-react for pre-built auth components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wardogordo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
