## linkurator-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Linkurator is a Next.js web application that helps users categorize their YouTube subscriptions into topics and view videos from channels in those categories. The frontend connects to a separate backend service for data and authentication.

## Development Commands

- `yarn dev` - Start development server (requires backend running on localhost:9000)
- `yarn build` - Build production version
- `yarn start` - Start production server
- `yarn lint` - Run ESLint
- `yarn test` - Run Jest tests
- `yarn coverage` - Run tests with coverage report
- `yarn install` - Install dependencies

## Architecture

### Tech Stack
- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS + DaisyUI components
- **State Management**: React Query (@tanstack/react-query) for server state
- **Internationalization**: next-intl
- **Testing**: Jest + React Testing Library
- **Type Safety**: TypeScript

### Project Structure

**Configuration**:
- `configuration.ts` - API endpoints and URL configuration (dev vs prod)
- Environment-aware: localhost:9000 for dev API, api.linkurator.com for prod

**Components** (Atomic Design Pattern):
- `components/atoms/` - Basic UI components (Button, Input, etc.)
- `components/molecules/` - Composite components (SearchBar, Card, etc.) 
- `components/organism/` - Complex feature components (modals, grids, etc.)

**Core Directories**:
- `app/` - Next.js App Router pages and layouts
- `entities/` - TypeScript interfaces/types
- `hooks/` - Custom React hooks for data fetching and state
- `services/` - API service functions
- `utilities/` - Helper functions
- `i18n/` - Internationalization config
- `messages/` - Translation files (en.json, es.json)

### Key Patterns

**Data Fetching**: All API calls use React Query hooks (prefix `use*`) that handle caching, loading states, and error handling.

**Routing**: Uses Next.js App Router with dynamic segments like `[[...id]]` for flexible routing.

**Styling**: Combines Tailwind utility classes with DaisyUI component classes.

**Authentication**: Integrates with backend OAuth flow via redirects to API endpoints.

## Development Notes

- Backend dependency: Requires [linkurator-backend](https://github.com/frantracer/linkurator-backend) running locally
- The app has example topic URLs hardcoded in configuration for demos
- Uses MSW (Mock Service Worker) for testing (public/mockServiceWorker.js)
- Deploys to Vercel on main branch pushes

## Backend API References

- Backend schema is available in http://localhost:9000/openapi.json

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frantracer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/frantracer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
