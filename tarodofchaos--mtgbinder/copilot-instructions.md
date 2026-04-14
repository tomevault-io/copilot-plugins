## mtgbinder

> MTG Binder is a full-stack Magic: The Gathering card collection and trading application. Users can:

# GEMINI.md - MTG Binder Project Guide

## Project Overview

MTG Binder is a full-stack Magic: The Gathering card collection and trading application. Users can:
- Track their card collections with condition, quantity, and trade availability
- Maintain wishlists with priority levels and price limits
- Share their trade binders via QR codes for public viewing
- Match trades with other users in real-time trading sessions

The application uses React (Vite) for the frontend, Express for the backend, and PostgreSQL with Prisma ORM for data persistence. Real-time features are powered by Socket.IO.

## Project Structure

```
├── client/                     # React frontend (Vite)
│   ├── src/
│   │   ├── components/
│   │   │   ├── cards/          # Card display components (CardImage, CardGrid, CardSearch)
│   │   │   ├── collection/     # Collection management (CollectionCard, AddCardForm)
│   │   │   ├── trading/        # Trading features (BinderView, PublicTradeCard, QRScanner)
│   │   │   ├── wishlist/       # Wishlist components
│   │   │   ├── layout/         # Header, BottomNav
│   │   │   └── ui/             # Reusable UI (LoadingSpinner, Modal)
│   │   ├── pages/              # Route-level pages
│   │   │   ├── CollectionPage.tsx
│   │   │   ├── WishlistPage.tsx
│   │   │   ├── TradePage.tsx
│   │   │   ├── TradeSessionPage.tsx
│   │   │   ├── PublicTradesPage.tsx  # Public binder view (/binder/:shareCode)
│   │   │   └── SearchPage.tsx
│   │   ├── services/           # API clients (api.ts, card-service.ts, etc.)
│   │   ├── context/            # React context (auth-context, theme-context)
│   │   ├── App.tsx             # Root component with routing
│   │   └── main.tsx            # Entry point
│   └── package.json
├── server/                     # Express backend
│   ├── src/
│   │   ├── routes/             # API route handlers
│   │   │   ├── auth.ts         # Authentication endpoints
│   │   │   ├── cards.ts        # Card search/autocomplete
│   │   │   ├── collection.ts   # Collection CRUD
│   │   │   ├── wishlist.ts     # Wishlist CRUD
│   │   │   ├── trade.ts        # Trade session management
│   │   │   └── binder.ts       # Public binder endpoints
│   │   ├── services/
│   │   │   ├── match-service.ts    # Trade matching algorithm (by card NAME)
│   │   │   └── socket-service.ts   # Socket.IO real-time events
│   │   ├── middleware/         # Auth, validation, error handling
│   │   ├── utils/              # Prisma client, logger, config
│   │   └── index.ts            # Server entry point
│   ├── prisma/
│   │   └── schema.prisma       # Database schema
│   ├── scripts/                # Utility scripts (add-test-trades.ts)
│   └── package.json
├── shared/                     # Shared types and utilities
│   └── src/index.ts            # Enums, interfaces, helper functions
├── docs/
│   └── architecture/decisions/ # ADRs (Architecture Decision Records)
└── package.json                # Root workspace configuration
```

## Key Features

### Public Binder Sharing
- Each user has a unique `shareCode` for their trade binder
- Public URL: `/binder/:shareCode` (e.g., `/binder/UG5ZH8N3`)
- QR code generation for easy sharing
- Two view modes: **Grid View** and **Binder View** (page-turning animation)
- Click any card to see enlarged detail popup with Cardmarket link

### Trade Matching
- Matches are based on card **NAME only** (not specific print/set)
- Compares User A's trade offers against User B's wishlist and vice versa
- Considers condition requirements and price limits from wishlists
- Real-time updates via Socket.IO during active trade sessions

### Card Data
- Uses Scryfall for card images: `https://cards.scryfall.io/{size}/front/{dir1}/{dir2}/{scryfallId}.jpg`
- Links to Cardmarket via search: `https://www.cardmarket.com/en/Magic/Products/Search?searchString={cardName}`
- Card conditions: M (Mint), NM (Near Mint), LP, MP, HP, DMG

## Common Commands

### Development
```bash
npm run dev              # Start both client (port 3000) and server (port 5000)
npm run dev:client       # Start Vite dev server only
npm run dev:server       # Start Express server only (tsx watch)
```

### Database
```bash
npm run db:migrate       # Run Prisma migrations
npm run db:generate      # Generate Prisma client
npm run db:seed          # Seed database with initial data
npm run import:cards     # Import card data from MTGJSON
```

### Testing
```bash
npm test                 # Run all tests (Vitest for client)
npm run test:coverage    # Generate coverage report
```

### Build & Production
```bash
npm run build            # Build shared, server, then client
npm start                # Start production server
```

### Code Quality
```bash
npm run lint             # Run ESLint across workspaces
npm run lint:fix         # Auto-fix lint issues
npm run typecheck        # Run TypeScript type checking
```

## Code Style Preferences

### General
- TypeScript for all code (client and server)
- Functional components with hooks (no class components)
- Named exports over default exports
- async/await over raw promises
- Keep functions small and focused

### Naming Conventions
- **Files**: kebab-case (`card-service.ts`), PascalCase for components (`CardGrid.tsx`)
- **Variables/Functions**: camelCase (`getUserById`, `isLoading`)
- **Constants**: UPPER_SNAKE_CASE (`CARDS_PER_PAGE`)
- **Types/Interfaces**: PascalCase (`CollectionItem`, `TradeMatch`)
- **Enums**: PascalCase with UPPER_SNAKE_CASE values (`CardCondition.NEAR_MINT`)

### React Patterns
- MUI (Material-UI v7) for styling via `sx` prop
- TanStack Query for server state management
- React Hook Form for form handling
- Socket.IO client for real-time features
- Co-locate component styles using `styles` object with `SxProps<Theme>`

### Express Patterns
- Router-level middleware for route groups
- Zod for request validation
- Prisma for database operations
- Winston for logging

## Things to Avoid

- **No `any` types** - Use `unknown` and narrow with type guards
- **No console.log in production** - Use the logger utility
- **No secrets in code** - Use environment variables
- **No business logic in route handlers** - Move to services
- **No direct DOM manipulation** - Use React refs when necessary
- **No barrel files** - Causes circular dependencies

## Key Dependencies

### Frontend (client)
| Package | Purpose |
|---------|---------|
| `react` + `react-dom` | UI library |
| `react-router-dom` | Client-side routing |
| `@mui/material` + `@mui/icons-material` | UI components |
| `@tanstack/react-query` | Server state and caching |
| `axios` | HTTP client |
| `socket.io-client` | Real-time communication |
| `react-hook-form` + `zod` | Form handling and validation |
| `qrcode` + `html5-qrcode` | QR code generation and scanning |

### Backend (server)
| Package | Purpose |
|---------|---------|
| `express` | Web framework |
| `@prisma/client` | Database ORM |
| `socket.io` | Real-time server |
| `jsonwebtoken` + `jscrypt` | Authentication |
| `zod` | Request validation |
| `winston` | Logging |
| `cors` + `helmet` | Security |

### Shared
| Export | Purpose |
|--------|---------|
| `CardCondition`, `WishlistPriority`, `TradeSessionStatus` | Enums |
| `Card`, `User`, `CollectionItem`, `WishlistItem`, `TradeMatch` | Interfaces |
| `getScryfallImageUrl()` | Build Scryfall image URLs |
| `getCardmarketUrl()` | Build Cardmarket search URLs |
| `isConditionAcceptable()` | Compare card conditions |

## Environment Variables

Required in `.env`:
```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://user:pass@localhost:5432/mtg_binder
JWT_SECRET=<secret-key>
CLIENT_URL=http://localhost:3000
```

## API Conventions

- RESTful endpoints under `/api/`
- Authentication via JWT Bearer token
- Response shape: `{ data?, error?, message? }`
- Pagination: `{ data[], total, page, pageSize, totalPages }`

### Key Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | Create account |
| `POST` | `/api/auth/login` | Get JWT token |
| `GET` | `/api/collection` | Get user's collection |
| `GET` | `/api/wishlist` | Get user's wishlist |
| `POST` | `/api/trade/sessions` | Create trade session |
| `GET` | `/api/binder/:shareCode` | Get public trade binder |

## Architecture Decisions

See `/docs/architecture/decisions/` for ADRs:
- **ADR-001**: Monolithic Architecture
- **ADR-002**: PostgreSQL as Primary Database
- **ADR-003**: Socket.IO for Real-Time Trading
- **ADR-004**: JWT for Authentication
- **ADR-005**: Trade Matching by Card Name Only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tarodofchaos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tarodofchaos)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
