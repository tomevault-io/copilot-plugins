## shopify-app-development-course

> - **Database**: SQLite for development (configured in [schema.prisma](mdc:prisma/schema.prisma))


# Prisma Database Patterns

## Database Setup
- **Database**: SQLite for development (configured in [schema.prisma](mdc:prisma/schema.prisma))
- **Client**: Singleton pattern in [db.server.js](mdc:app/db.server.js)
- **Session Storage**: Shopify sessions managed automatically

## Schema Patterns
```prisma
model Session {
  id            String    @id
  shop          String    // Shopify shop domain
  state         String    // OAuth state
  isOnline      Boolean   @default(false)
  scope         String?   // App permissions
  expires       DateTime?
  accessToken   String    // Shop access token
  userId        BigInt?   // Shopify user ID
  // ... other user fields
}
```

## Database Client Usage
```jsx
import prisma from "../db.server";

// In route loaders/actions
export const loader = async ({ request }) => {
  await authenticate.admin(request);

  // Use Prisma client
  const records = await prisma.yourModel.findMany();
  return { records };
};
```

## Migration Commands
- `npx prisma migrate dev --name description`: Create and apply new migration
- `npx prisma migrate deploy`: Apply pending migrations (production)
- `npx prisma db push`: Push schema changes without migration (development)
- `npx prisma generate`: Generate client after schema changes

## Best Practices
1. **Always migrate**: Don't use `db push` in production
2. **Descriptive names**: Use clear migration names
3. **Schema first**: Define schema, then generate client
4. **Environment safety**: Different databases for dev/staging/production
5. **Backup before migrations**: Especially in production

## Custom Models Pattern
```prisma
model Product {
  id          String   @id @default(cuid())
  shopifyId   String   @unique  // Store Shopify GID
  title       String
  handle      String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  shop        String   // Link to shop domain

  @@map("products")
}
```

## Connection Management
- Prisma client is automatically managed
- Connection pooling handled by Prisma
- Use singleton pattern to avoid multiple instances
- Close connections in serverless environments if needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LazerTechnologies)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LazerTechnologies)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
