## catatan-uang

> The application uses Prisma ORM with PostgreSQL. Key models include:

# Data Flow

## Database Schema

The application uses Prisma ORM with PostgreSQL. Key models include:
- User: Authentication and user data
- Transaction: Financial transactions
- Customer: Customer profiles
- Report: Generated financial reports

Schema definition: [prisma/schema.prisma](mdc:prisma/schema.prisma)

## API Structure

API endpoints follow RESTful patterns in `/pages/api`:

- [pages/api/transaksi.js](mdc:pages/api/transaksi.js): CRUD operations for transactions
- [pages/api/customer.js](mdc:pages/api/customer.js): Customer management endpoints
- [pages/api/laporan.js](mdc:pages/api/laporan.js): Report generation and retrieval
- [pages/api/auth](mdc:pages/api/auth): Authentication endpoints using NextAuth.js

## Data Handling

1. Frontend components make fetch requests to API endpoints
2. API routes use Prisma client for database operations
3. Response data is rendered in the appropriate components
4. Forms use React state for local data management before submission

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AdiDwiww) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
