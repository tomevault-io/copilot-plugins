## ghl-de-duper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GoHighLevel Contact De-Duper - A React application that identifies and merges duplicate contacts in GoHighLevel CRM. The application uses OAuth to authenticate with GoHighLevel and provides a web interface for contact deduplication.

## Architecture

**Frontend (React + TypeScript + Vite):**
- React Router for navigation with protected routes
- Auth context managing user state and GHL OAuth flow
- Settings context for application configuration
- Component structure: Layout wraps all pages, ProtectedRoute guards authenticated areas
- Main pages: HomePage (marketing), DashboardPage, DuplicatesPage, MergePage, HistoryPage

**Backend (Node.js + Fastify):**
- Fastify server on port 3000 with CORS enabled
- Integrates with GoHighLevel API for contact operations
- Supabase integration for logging merge activities
- Duplicate detection using email/phone fuzzy matching with Levenshtein distance
- Key endpoints: `/contacts/search`, `/contacts/merge`

**Database:**
- Supabase for storing merge logs and user data
- Migration files in `supabase/migrations/`

## Development Commands

```bash
# Start frontend development server
npm run dev

# Start backend server (must run separately)
npm run dev:backend

# Build for production
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Environment Setup

Required environment variables:
- `VITE_SUPABASE_URL` - Supabase project URL
- `VITE_SUPABASE_SERVICE_ROLE_KEY` - Supabase service role key
- `VITE_GHL_CLIENT_ID` - GoHighLevel OAuth client ID

## Key Implementation Details

**OAuth Flow:**
- Initiated via `AuthContext.initiateGHLAuth()` 
- Callback handled in `OAuthCallback` page component
- Tokens stored in localStorage and attached to user context

**Contact Processing:**
- Phone numbers normalized to E.164 format in `normalizePhone()` function
- Duplicate detection in `findDuplicates()` using email/phone similarity
- Merge operations preserve tags and notes based on user preferences

**API Proxy:**
- Frontend `/api` requests proxied to `localhost:3000` via Vite config
- Backend handles GHL API authentication and rate limiting

**State Management:**
- AuthContext manages user authentication and GHL OAuth state
- SettingsContext handles application preferences
- No external state management library used

---
> Source: [luissanchez0305/ghl-de-duper](https://github.com/luissanchez0305/ghl-de-duper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
