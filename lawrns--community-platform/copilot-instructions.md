## community-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a community platform with features for content creation, user reputation, tool directory, and more. The project uses a Next.js frontend with a Node.js/Express backend API integrated with Supabase.

## Key Commands

### Frontend Development
```bash
# Install dependencies
cd frontend
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test
```

### Backend Development
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Run tests
npm test

# Run specific test file
npm test -- src/tests/path/to/test.js

# Migrate database
npm run migrate
```

## Architecture Overview

### Frontend
- Next.js application with app router pattern
- UI components using shadcn/ui and custom components
- State management via React context
- API integration through the centralized api.ts client

### Backend
- Express.js API server
- Repository pattern for database access
- Authentication via JWT and WebAuthn
- Integration with Supabase for database and vector search
- WebSocket support for real-time notifications

### Key Features
1. **Authentication System**: Email/password and passwordless WebAuthn
2. **Content Management**: Rich text editor with versioning
3. **Reputation System**: Points, badges, and privileges
4. **Tool Directory**: Listings, reviews, and vendor claims
5. **Search Engine**: Vector and lexical search
6. **Quadratic Voting**: Credit-based weighted voting
7. **Daily Briefs**: Personalized content digests

## Code Patterns

### Frontend Component Structure
- UI components in `/frontend/components/ui`
- Feature components in `/frontend/components/{feature}`
- Page components in `/frontend/app/{route}`
- API client in `/frontend/lib/api.ts`

### Backend API Structure
- Routes defined in `/src/routes/api/{feature}`
- Models in `/src/models`
- Services in `/src/services/{feature}`
- Middlewares in `/src/middlewares`

### Database
- Supabase PostgreSQL with pgvector extension
- Migrations in `/supabase/migrations`
- Repository pattern with models in `/src/models`

## UI/UX Integration Status

The project is currently focusing on Frontend-Backend Integration (Task 12). Key integration tasks include:

1. Connecting rich text editor to content API
2. Implementing search results with real backend data
3. Setting up WebSocket for real-time notifications
4. Connecting tool listings to backend API
5. Implementing reputation display in profiles
6. Integrating WebAuthn components with backend
7. Implementing quadratic voting UI with API integration
8. Connecting the daily briefs feature to backend

## Environment Configuration

The project uses environment variables for configuration:

- `NEXT_PUBLIC_API_URL`: Backend API URL for frontend 
- `DATABASE_URL`: Supabase PostgreSQL connection string
- `JWT_SECRET`: Secret for JWT token generation
- `SUPABASE_URL`: Supabase project URL
- `SUPABASE_KEY`: Supabase service key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lawrns)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lawrns)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
