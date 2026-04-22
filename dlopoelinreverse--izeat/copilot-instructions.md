## izeat

> - Next.js 16.0.5 + React 19.2.0

# izEat - Restaurant Management SaaS

## Tech Stack

### Frontend (web/)
- Next.js 16.0.5 + React 19.2.0
- Apollo Client 4.0.9 (GraphQL)
- Tailwind CSS 4 + Radix UI
- Better Auth (authentication)
- TypeScript 5

### Backend (backend/)
- Express 5.1.0 + Apollo Server 5.2.0
- TypeGraphQL 2.0.0-rc.2
- TypeORM 0.3.27
- PostgreSQL 16
- Better Auth

## Project Structure

```
├── backend/src/
│   ├── index.ts           # Apollo Server entry (port 4001)
│   ├── schema.ts          # GraphQL schema builder
│   ├── data-base.ts       # TypeORM config
│   ├── auth.ts            # Auth checker
│   ├── entities/          # 11 TypeORM entities
│   ├── resolvers/         # 10 GraphQL resolvers
│   ├── migrations/        # DB migrations
│   └── lib/               # Better-auth config
│
├── web/
│   ├── app/
│   │   ├── page.tsx       # Landing page
│   │   ├── providers.tsx  # Apollo + Theme providers
│   │   ├── auth/          # Sign-in/Sign-up pages
│   │   └── app/dashboard/ # Main app dashboard
│   │       ├── page.tsx   # Restaurant creation
│   │       └── [restaurantId]/
│   │           ├── ingredients/
│   │           ├── menus/
│   │           ├── tables/
│   │           └── service/
│   ├── components/
│   │   ├── dashboard/     # Feature components
│   │   └── ui/            # Radix UI primitives
│   ├── contexts/          # React contexts
│   ├── graphql/
│   │   ├── query/         # GraphQL queries
│   │   ├── mutations/     # GraphQL mutations
│   │   └── __generated__/ # CodeGen types
│   └── hooks/             # Custom hooks
│
└── compose.yml            # Docker Compose
```

## Database Entities

| Entity | Description |
|--------|-------------|
| User | Authenticated users |
| Session | Auth sessions |
| Account | OAuth accounts |
| Restaurant | User-owned restaurants |
| Menu | Restaurant menus |
| MenuItem | Menu items with pricing |
| MenuCategory | Menu item categories |
| MenuItemIngredient | Item-ingredient junction |
| Ingredient | Restaurant ingredients |
| IngredientCategory | Ingredient groups |
| RestaurantTable | Tables with capacity/status |

## GraphQL

- **Endpoint:** `http://localhost:4001/graphql`
- **Schema:** Built with TypeGraphQL decorators
- **Auth:** `@Authorized()` decorator on protected resolvers
- **Context:** User from Better Auth session

## Commands

```bash
# Development
npm run dev              # Start both backend + frontend

# Docker
docker compose up        # Run containerized

# Code generation (frontend)
cd web && npm run codegen
```

## Conventions

- UUID primary keys on all entities
- Timestamps (createdAt, updatedAt) on key entities
- CASCADE delete for related data
- Restaurant-scoped data access (user owns restaurant)
- TypeGraphQL decorators for schema definition

## Features

### Onboarding Flow (5 steps)
1. Restaurant creation
2. Menu creation
3. Menu categories
4. Menu items (dishes)
5. Table setup

Status tracked via `me` query → `onboarding` field

### Multi-Restaurant Support
Routes use `[restaurantId]` dynamic segment for restaurant-scoped views

## Key Files

- Backend entry: `backend/src/index.ts`
- DB config: `backend/src/data-base.ts`
- Auth config: `backend/src/lib/auth.ts`
- Frontend providers: `web/app/providers.tsx`
- Apollo client: `web/lib/apollo-client.ts`
- Onboarding context: `web/contexts/onboarding-context.tsx`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dlopoelinreverse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
