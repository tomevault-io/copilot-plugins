## chat-api

> import { PrismaClient } from '@prisma/client'


# Prisma ORM Standards

## Prisma Client Singleton

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  })

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

## Query Patterns

```typescript
// Include relations một cách an toàn
const conversation = await prisma.conversation.findUnique({
  where: { id: conversationId },
  include: {
    participants: {
      include: { user: true },
    },
    messages: {
      orderBy: { createdAt: 'desc' },
      take: 20,
    },
  },
})

// Soft delete
await prisma.conversation.update({
  where: { id },
  data: { deletedAt: new Date() },
})

// Transaction cho complex operations
await prisma.$transaction(async (tx) => {
  const message = await tx.message.create({
    data: { conversationId, senderId, content, type: 'TEXT' },
  })
  
  await tx.conversation.update({
    where: { id: conversationId },
    data: { lastMessageId: message.id, updatedAt: new Date() },
  })
  
  return message
})
```

## Pagination với Cursor

```typescript
// src/services/messageService.ts
interface PaginationParams {
  cursor?: string
  limit: number
}

async getMessages(conversationId: string, params: PaginationParams) {
  const { cursor, limit = 20 } = params
  
  const messages = await prisma.message.findMany({
    where: { 
      conversationId,
      isDeleted: false,
      ...(cursor ? { createdAt: { lt: new Date(cursor) } } : {}),
    },
    orderBy: { createdAt: 'desc' },
    take: limit + 1,
    include: {
      sender: { select: { id: true, fullName: true, avatarUrl: true } },
      parent: true,
    },
  })

  const hasMore = messages.length > limit
  const items = hasMore ? messages.slice(0, -1) : messages
  const nextCursor = hasMore ? items[items.length - 1].createdAt.toISOString() : null

  return { items, nextCursor, hasMore }
}
```

## Type Generation

```bash
# Sau khi thay đổi schema, chạy:
npx prisma generate

# Migrate database:
npx prisma migrate dev --name add_feature

# Push schema (dev only):
npx prisma db push
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skytam1234) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
