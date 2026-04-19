## tsundoc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
# Root level (runs both frontend and backend)
pnpm dev

# Backend only
cd backend && make dev    # Includes code generation + hot reload

# Frontend only  
cd frontend/app/web && pnpm dev
```

### Code Generation
```bash
# Backend: Generate GraphQL resolvers from schema
cd backend && make generate

# Frontend: Generate TypeScript types from GraphQL operations
cd frontend/app/web && pnpm codegen
```

### Build & Test
```bash
# Build everything
pnpm build

# Run tests
pnpm test

# Lint code
pnpm lint

# TypeScript type checking (frontend)
cd frontend/app/web && pnpm typecheck
```

### Docker & Database
```bash
# Start PostgreSQL
docker-compose up -d postgres

# Start all services (PostgreSQL + Backend)
docker-compose up -d

# View logs
docker-compose logs -f
```

## Architecture

This is a monorepo with separate backend (Go) and frontend (Next.js) applications connected via GraphQL.

### Backend (Domain-Driven Design)
- **cmd/server/**: Entry points (main.go for GraphQL, simple.go for basic HTTP)
- **internal/domain/**: Domain entities (Book)
- **internal/interface/graphql/**: GraphQL resolvers and generated code
- **internal/usecase/**: Business logic (to be implemented)
- **internal/infra/**: External services integration (to be implemented)
- **graphql/schema/**: GraphQL schema definitions

### Frontend (Next.js App Router)
- **src/app/**: Pages using App Router (library, save)
- **src/components/**: React components organized by feature
- **src/lib/graphql/**: GraphQL client, queries, and mutations
- **src/lib/firebase/**: Firebase Auth configuration

### Key Technologies
- **Backend**: Go with gqlgen for GraphQL, Chi router, Firebase Auth middleware
- **Frontend**: Next.js 14, TypeScript, Tailwind CSS, graphql-request
- **Database**: PostgreSQL 16
- **Authentication**: Firebase Auth on both frontend and backend
- **Package Management**: pnpm workspace (monorepo)

### GraphQL API
The API is defined in `backend/graphql/schema/schema.graphqls` with:
- **Types**: Book (with AI-generated title, tags, and content)
- **Queries**: book(id), myBooks(keyword)  
- **Mutations**: saveBook, updateBook, mergeBooks

Always run code generation after modifying GraphQL schema or operations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/motoya-k) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
