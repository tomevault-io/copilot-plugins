## database

> Padrões e convenções para banco de dados e Drizzle ORM


# Padrões de Banco de Dados

## Stack

- **Database**: Neon PostgreSQL (serverless)
- **ORM**: Drizzle ORM
- **Adapter**: `drizzle-orm/neon-http` (HTTP adapter para serverless)
- **Cliente**: `@neondatabase/serverless`

## Conexão

### Arquivo: `src/db/index.ts`

```typescript
import { neon } from "@neondatabase/serverless"
import { drizzle } from "drizzle-orm/neon-http"

const sql = neon(process.env.DATABASE_URL!)
export const db = drizzle({ client: sql })
```

### Variável de Ambiente
```env
DATABASE_URL=postgresql://user:password@host/database?sslmode=require
```

## Schema

### Arquivo: `src/db/schema.ts`

### Estrutura de Tabelas (8 Tabelas)

#### 1. users
- **ID**: Clerk user ID (text)
- **Campos**: email, name, avatarUrl, timestamps, deletedAt (soft delete)
- **Índices**: email, created_at

#### 2. chats
- **Relacionamento**: belongsTo users
- **Campos**: title, model (OpenRouter model), timestamps
- **Índices**: user_id, created_at

#### 3. messages
- **Relacionamento**: belongsTo chats
- **Campos**: role (user/assistant/system), content, createdAt
- **Índices**: chat_id, created_at

#### 4. library_items
- **Relacionamento**: belongsTo users, hasMany scheduled_posts
- **Campos**: type (enum), status (enum), title, content (JSON), mediaUrl, metadata, scheduledFor, publishedAt
- **Índices**: user_id, status, type, scheduled_for

#### 5. documents
- **Relacionamento**: belongsTo users
- **Campos**: title, content, sourceUrl, fileType, metadata
- **Índices**: user_id, created_at

#### 6. sources
- **Relacionamento**: belongsTo users
- **Campos**: name, url, type, config (JSON), lastScrapedAt, isActive
- **Índices**: user_id, unique(user_id, url)

#### 7. scheduled_posts
- **Relacionamento**: belongsTo library_items
- **Campos**: platform, scheduledFor, status, postedAt, platformPostId, error
- **Índices**: scheduled_for, status

#### 8. jobs
- **Relacionamento**: belongsTo users
- **Campos**: type (enum), status (enum), payload (JSONB), result (JSONB), error, priority, attempts, maxAttempts, timestamps
- **Índices**: user_id, status, type, scheduled_for, created_at

### Enums

```typescript
// Content Status
"draft" | "scheduled" | "published" | "archived"

// Post Type
"text" | "image" | "carousel" | "video" | "story"

// Job Type
"ai_text_generation" | "ai_image_generation" | "carousel_creation" | "scheduled_publish" | "web_scraping"

// Job Status
"pending" | "processing" | "completed" | "failed"
```

### Relations (Drizzle)

Todas as tabelas têm relations definidas:
- `usersRelations` - hasMany: chats, libraryItems, documents, sources, jobs
- `chatsRelations` - belongsTo: user, hasMany: messages
- `messagesRelations` - belongsTo: chat
- `libraryItemsRelations` - belongsTo: user, hasMany: scheduledPosts
- `documentsRelations` - belongsTo: user
- `sourcesRelations` - belongsTo: user
- `scheduledPostsRelations` - belongsTo: libraryItem
- `jobsRelations` - belongsTo: user

### Type Exports

```typescript
// Infer types from schema
export type User = typeof users.$inferSelect
export type NewUser = typeof users.$inferInsert
// ... (para todas as tabelas)
```

## Queries

### Padrão de Query

```typescript
import { db } from "@/db"
import { users, chats } from "@/db/schema"
import { eq, desc, and } from "drizzle-orm"

// SELECT simples
const user = await db.select().from(users).where(eq(users.id, userId)).limit(1)

// SELECT com relacionamento
const chatsWithUser = await db
  .select()
  .from(chats)
  .innerJoin(users, eq(chats.userId, users.id))
  .where(eq(chats.userId, userId))
  .orderBy(desc(chats.createdAt))

// INSERT
const [newChat] = await db
  .insert(chats)
  .values({ userId, title, model })
  .returning({ id: chats.id })

// UPDATE
await db
  .update(chats)
  .set({ title: "New Title" })
  .where(eq(chats.id, chatId))

// DELETE (soft delete)
await db
  .update(users)
  .set({ deletedAt: new Date() })
  .where(eq(users.id, userId))
```

### Operadores Comuns

```typescript
import { eq, ne, gt, gte, lt, lte, like, and, or, not, inArray } from "drizzle-orm"

// Igualdade
eq(users.id, userId)

// Múltiplas condições
and(eq(users.id, userId), eq(chats.status, "active"))

// Ordenação
orderBy(desc(chats.createdAt), asc(chats.title))

// Limite e offset
.limit(10).offset(20)
```

## Migrations

### Configuração: `drizzle.config.ts`

```typescript
export default defineConfig({
  schema: "./src/db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
})
```

### Comandos

```bash
# Gerar migration baseada em mudanças no schema
npm run db:generate

# Executar migrations
npm run db:migrate

# Push schema diretamente (sem migration)
npm run db:push

# Studio visual (UI do banco)
npm run db:studio

# Introspect (gerar schema a partir do banco)
npm run db:pull
```

### Workflow de Migration

1. **Modificar schema** em `src/db/schema.ts`
2. **Gerar migration**: `npm run db:generate`
3. **Revisar** arquivos em `drizzle/`
4. **Executar migration**: `npm run db:migrate`
5. **Verificar** com `npm run db:studio`

## Padrões de Dados

### Soft Delete
- Campos `deletedAt: timestamp("deleted_at")`
- Não deletar fisicamente, apenas marcar como deletado
- Filtrar em queries: `where(isNull(users.deletedAt))`

### Timestamps
- `createdAt`: `timestamp("created_at").defaultNow().notNull()`
- `updatedAt`: `timestamp("updated_at").defaultNow().notNull()`
- Atualizar `updatedAt` manualmente em updates

### JSON Fields
- **payload/result**: `jsonb("payload").$type<Record<string, unknown>>()`
- **metadata**: `text("metadata")` (armazenar como JSON string)
- **content**: `text("content")` (JSON string para conteúdo estruturado)

### Foreign Keys
- Sempre usar `references()` com `onDelete: "cascade"` quando apropriado
- Índices em foreign keys para performance

## Performance

### Índices
- Criar índices em:
  - Foreign keys (`user_id`, `chat_id`, etc)
  - Campos de filtro frequente (`status`, `type`, `scheduled_for`)
  - Campos de ordenação (`created_at`)

### Queries Otimizadas
- Usar `select()` com campos específicos quando possível
- Evitar `select().from()` sem `where()` em tabelas grandes
- Usar `limit()` e `offset()` para paginação

### Connection Pooling
- Neon gerencia pooling automaticamente
- HTTP adapter é otimizado para serverless

## Segurança

### Row Level Security (RLS)
- Implementar RLS no Neon quando necessário
- Filtrar por `userId` em todas as queries de usuário

### SQL Injection
- Drizzle previne SQL injection automaticamente
- Nunca concatenar strings em queries
- Sempre usar operadores do Drizzle (`eq`, `and`, etc)

### Validação
- Validar dados antes de inserir/atualizar
- Usar Zod ou similar para validação de schemas

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
