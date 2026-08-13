## monorepo

> This file provides guidance for Cursor AI when working with the Krain monorepo. It defines the project structure, technology stack (Next.js, React, TypeScript, Tailwind CSS, Drizzle ORM), and coding standards to ensure consistent code generation and suggestions. The document helps Cursor understand the monorepo architecture with apps and packages, follow established patterns for components, database operations, and authentication, and maintain the project's coding style and organization.

# Krain Project Rules

## Project Structure
- This is a monorepo using Turborepo and pnpm workspaces
- The project is organized into `apps/` and `packages/` directories
- Apps include: airdrop, landing, token, marketplace, early
- Packages include: ui, db, utils, session, typescript-config, eslint-config

## Technology Stack
- Next.js (v15+) for frontend applications
- React (v19+) for UI components
- TypeScript (v5+) for type safety
- Tailwind (v4) CSS for styling
- Drizzle ORM for database operations
- Privy for authentication
- Solana Web3.js for blockchain interactions
- Shadcn/ui in the local `packages/ui` for UI components
- Vercel for deployment

## Coding Standards

### General
- Use TypeScript for all code
- Follow ESLint configuration provided in the eslint-config package
- Use Prettier for code formatting
- Use named exports instead of default exports
- Use functional components with hooks for React

### File Structure
- React components should be in PascalCase
- Utility functions should be in camelCase
- Use `.tsx` extension for React components
- Use `.ts` extension for TypeScript files
- Place shared UI components in the `packages/ui` directory
- Place database schema and operations in the `packages/db` directory
- Place utility functions in the `packages/utils` directory

### Component Structure
- Use the shadcn/ui pattern for UI components
- Use Tailwind CSS for styling
- Use Radix UI for accessible components
- Use the `cva` (class-variance-authority) for component variants
- Use the `cn` utility for merging Tailwind classes

### Database
- Use Drizzle ORM for database operations
- Define schemas in the `packages/db/schema.ts` file
- Use the database client from `packages/db/client.ts`

### State Management
- Use React hooks for local state
- Use React Context for global state when necessary
- Use server actions for data mutations

### Authentication
- Use Privy for authentication
- Use the `@privy-io/react-auth` package
- Handle wallet connections through Privy

### API
- Use Next.js API routes for backend functionality
- Use server actions for data mutations
- Follow RESTful principles for API design

### Error Handling
- Use try/catch blocks for error handling
- Use toast notifications for user feedback
- Log errors appropriately

### Testing
- Write unit tests for utility functions
- Write integration tests for components
- Use Next.js testing utilities for page tests

## Workflow
- Use pnpm for package management
- Use Turborepo for monorepo management
- Use `pnpm dev` to start the development server
- Use `pnpm build` to build the project
- Use `pnpm lint` to lint the project
- Use `pnpm clean` to clean the project

## Deployment
- Use Vercel for deployment
- Configure environment variables in Vercel
- Use Vercel KV for caching and rate limiting
- Use Vercel Postgres for database

## Environment Variables
- Store environment variables in `.env` files
- Use `.env.example` files for documentation
- Reference environment variables in `turbo.json` for global access

---
> Source: [jesus270/Krain-MVP-v1](https://github.com/jesus270/Krain-MVP-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
