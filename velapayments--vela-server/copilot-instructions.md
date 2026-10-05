## database

> database


# Database Setup Guide

This guide explains how to set up and use Prisma with Supabase in the Core API.

## 📋 Prerequisites

- Supabase account and project
- Node.js installed
- Dependencies installed (`npm install`)

## 🚀 Initial Setup

### 1. Get Supabase Connection Strings

1. Go to your Supabase project dashboard
2. Navigate to **Project Settings > Database**
3. Copy the connection strings:
   - **Connection pooling** (for DATABASE_URL)
   - **Direct connection** (for DIRECT_URL)

### 2. Configure Environment Variables

Create a `.env` file in the root directory:

```bash
# Copy the example file
cp .env.example .env
```

Then edit `.env` and add your Supabase credentials:

```env
DATABASE_URL="postgresql://postgres:[YOUR-PASSWORD]@[YOUR-PROJECT-REF].pooler.supabase.com:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://postgres:[YOUR-PASSWORD]@[YOUR-PROJECT-REF].supabase.com:5432/postgres"
```

### 3. Update Prisma Schema

Edit `prisma/schema.prisma` with your actual data models. The file currently has an example `User` model.

### 4. Generate Prisma Client

```bash
npm run prisma:generate
```

This generates the Prisma Client based on your schema.

### 5. Push Schema to Database

For development (no migration files):

```bash
npm run prisma:push
```

Or create a migration (recommended for production):

```bash
npm run prisma:migrate
```

## 📚 Available Commands

| Command                   | Description                    |
| ------------------------- | ------------------------------ |
| `npm run prisma:generate` | Generate Prisma Client         |
| `npm run prisma:migrate`  | Create and apply migrations    |
| `npm run prisma:push`     | Push schema changes (dev only) |
| `npm run prisma:studio`   | Open Prisma Studio (GUI)       |
| `npm run prisma:seed`     | Seed the database              |

## 🏗️ Architecture

### Clean Code Structure

```
src/
├── database/
│   ├── database.module.ts    # Global database module
│   ├── prisma.service.ts     # Prisma service with lifecycle hooks
│   └── index.ts              # Barrel export
├── modules/
│   └── [feature]/
│       ├── [feature].module.ts
│       ├── [feature].service.ts
│       ├── [feature].controller.ts
│       └── dto/
└── app.module.ts
```

### Using Prisma in Your Services

The `PrismaService` is globally available. Just inject it:

```typescript
import { Injectable } from "@nestjs/common";
import { PrismaService } from "../database";

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll() {
    return this.prisma.user.findMany();
  }

  async findOne(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
    });
  }

  async create(data: CreateUserDto) {
    return this.prisma.user.create({
      data,
    });
  }

  async update(id: string, data: UpdateUserDto) {
    return this.prisma.user.update({
      where: { id },
      data,
    });
  }

  async remove(id: string) {
    return this.prisma.user.delete({
      where: { id },
    });
  }
}
```

## 🔍 Prisma Studio

To visually explore and edit your database:

```bash
npm run prisma:studio
```

This opens a web interface at `http://localhost:5555`

## 🌱 Seeding

To populate your database with initial data:

1. Edit `prisma/seed.ts` with your seed logic
2. Run: `npm run prisma:seed`

## 🔄 Migrations Workflow

### Development

```bash
# Make changes to schema.prisma
npm run prisma:migrate
# Name your migration when prompted
```

### Production

```bash
# Apply pending migrations
npx prisma migrate deploy
```

## 🛡️ Best Practices

1. **Always use transactions** for multiple related operations
2. **Use connection pooling** (DATABASE_URL with pgbouncer)
3. **Use direct connection** (DIRECT_URL) only for migrations
4. **Never commit** `.env` files
5. **Use DTOs** for validation before database operations
6. **Handle errors** properly with try-catch blocks
7. **Use indexes** for frequently queried fields

## 🔒 Row Level Security (RLS)

Row Level Security is enabled via a SQL script that you run once in Supabase. It protects data when access goes through the Supabase Data API (anon/authenticated keys). The Core API uses Prisma with a role that typically bypasses RLS, so authorization remains enforced in NestJS guards and services; RLS adds defense in depth.

**To apply RLS:**

1. Open your Supabase project dashboard.
2. Go to **SQL Editor**.
3. Open `prisma/supabase-rls.sql` from this repo, copy its contents, and run it.

The script will:

- Create a `private` schema and helper functions (`current_user_has_role`, `current_user_is_project_maintainer`).
- Enable RLS on all public tables.
- Add policies for user-scoped tables (user, kyc, profiles, wallet, notification): users can only see/edit their own rows.
- Add policies for project and project_maintainer (and project_review for admins).
- Add read-only policies for catalog tables (category, subcategory, tier_definition).
- Leave backend-only tables (campaign, escrow, email_job, etc.) with RLS on and no policies, so only roles with BYPASSRLS (e.g. Prisma) can access them.

The logic of each policy is documented in [ROW_LEVEL_SECURITY.md](./ROW_LEVEL_SECURITY.md). See also [Supabase Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security) for details.

## 🔐 Security Notes

- Never expose your database credentials
- Use environment variables for all sensitive data
- Enable Row Level Security (RLS) in Supabase using the script above
- Use Supabase's built-in auth when possible

## 📖 Resources

- [Prisma Documentation](https://www.prisma.io/docs)
- [Supabase Documentation](https://supabase.com/docs)
- [NestJS Prisma Integration](https://docs.nestjs.com/recipes/prisma)

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
