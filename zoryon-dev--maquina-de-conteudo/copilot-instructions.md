## architecture

> Arquitetura e padrões arquiteturais do projeto


# Arquitetura do Projeto

## Padrão Arquitetural

**Next.js App Router** com separação clara entre:
- **Server Components** (padrão) - Data fetching, lógica de servidor
- **Client Components** (apenas quando necessário) - Interatividade, hooks, browser APIs

## Estrutura de Rotas

### Rotas Públicas
- `/` - Landing page (redireciona autenticados para `/dashboard`)
- `/sign-in` - Login
- `/sign-up` - Registro
- `/api/webhooks/*` - Webhooks externos

### Rotas Protegidas (Grupo `(app)`)
Todas as rotas dentro de `src/app/(app)/` requerem autenticação:
- `/dashboard` - Chat com IA
- `/library` - Biblioteca de conteúdo
- `/calendar` - Calendário de posts
- `/sources` - Fontes de conteúdo
- `/settings` - Configurações

**Proteção**: `src/app/(app)/layout.tsx` verifica `auth()` e redireciona não autenticados.

## Middleware (Proteção de Rotas)

**Arquivo**: `src/proxy.ts`

```typescript
// Usa clerkMiddleware() para proteger rotas
// Redireciona usuários autenticados de "/" para "/dashboard"
// Protege todas as rotas em (app)/
```

## Banco de Dados

### Schema (8 Tabelas)

1. **users** - Sincronizado com Clerk (id = Clerk user ID)
2. **chats** - Threads de conversa com IA
3. **messages** - Mensagens individuais (user/assistant/system)
4. **library_items** - Biblioteca de conteúdo (text, image, carousel, etc)
5. **documents** - Base de conhecimento (upload de arquivos)
6. **sources** - Fontes de conteúdo para scraping
7. **scheduled_posts** - Fila de publicação agendada
8. **jobs** - Background jobs (geração de conteúdo, scraping, etc)

### Conexão
- **Neon Serverless** via `@neondatabase/serverless`
- **Drizzle ORM** com adapter `drizzle-orm/neon-http`
- Schema em `src/db/schema.ts`
- Cliente em `src/db/index.ts`

### Migrations
- Gerar: `npm run db:generate`
- Executar: `npm run db:migrate`
- Studio: `npm run db:studio`

## Sistema de Filas

### Arquitetura Serverless
- **Upstash Redis** (HTTP REST API) - Sem worker process contínuo
- **Fila**: Lista Redis (`jobs:pending`)
- **Workers**: API Routes (`/api/workers`) - Chamadas sob demanda
- **Persistência**: Tabela `jobs` no PostgreSQL

### Fluxo de Processamento

```
1. Client → POST /api/jobs
   ↓
2. Cria job no DB (status: pending)
   ↓
3. Enfileira no Redis (enqueueJob)
   ↓
4. Agendador/Cron → POST /api/workers
   ↓
5. Worker desenfileira (dequeueJob)
   ↓
6. Processa job (handlers por tipo)
   ↓
7. Atualiza DB (status: completed/failed)
   ↓
8. Remove da fila de processamento
```

### Tipos de Jobs

```typescript
type JobType =
  | "ai_text_generation"
  | "ai_image_generation"
  | "carousel_creation"
  | "scheduled_publish"
  | "web_scraping"
```

### Estrutura de Arquivos

```
src/lib/queue/
├── types.ts      # JobType, JobStatus, JobPayload
├── client.ts     # Redis client (enqueue, dequeue)
└── jobs.ts       # CRUD de jobs (createJob, updateJobStatus)
```

## Autenticação (Clerk)

### Configuração
- **Provider**: `<ClerkProvider>` em `src/app/layout.tsx`
- **Middleware**: `clerkMiddleware()` em `src/proxy.ts`
- **Proteção**: `auth()` de `@clerk/nextjs/server` em layouts

### Sincronização de Usuários
- **Webhook**: `/api/webhooks/clerk`
- Eventos: `user.created`, `user.updated`, `user.deleted`
- Sincroniza com tabela `users` no banco

### Componentes de Auth
- `src/components/auth/sign-in-card.tsx`
- `src/components/auth/sign-up-card.tsx`
- `src/components/auth/user-menu.tsx`
- `src/components/auth/oauth-buttons.tsx`

## Componentes

### Layout Principal
- **AppLayout** (`src/components/app-layout.tsx`):
  - Header fixo com glassmorphism
  - Navbar animada (tubelight)
  - Logo com hover effects
  - UserMenu no canto direito

### Componentes UI
- Base: **shadcn/ui** (New York style)
- Localização: `src/components/ui/`
- Padrão: Server Components quando possível

### Componentes de Feature
- `src/components/dashboard/animated-ai-chat.tsx` - Chat com IA
- `src/components/chat/model-selector.tsx` - Seletor de modelos

## State Management

### Zustand Stores
- Localização: `src/stores/` (futuro)
- Uso: Estado global do cliente (não server state)

### Server State
- **Data fetching**: Server Components com `async/await`
- **Mutations**: Server Actions (futuro)
- **Cache**: Next.js cache + revalidation

## API Routes

### Estrutura
```
src/app/api/
├── jobs/
│   ├── route.ts          # POST (criar), GET (listar)
│   └── [id]/route.ts     # GET (detalhes), DELETE
├── workers/
│   └── route.ts          # POST (processar job)
└── webhooks/
    └── clerk/route.ts    # POST (webhook Clerk)
```

### Padrões
- Autenticação: `auth()` de Clerk
- Respostas: `NextResponse.json()`
- Erros: Status codes apropriados (400, 401, 404, 500)

## Modelos de IA (OpenRouter)

### Configuração
- **Arquivo**: `src/lib/models.ts`
- **Provider**: OpenRouter (multi-modelo)
- **Tipos**: Text e Image

### Modelos Disponíveis
- **Texto**: GPT 5.2, Claude Sonnet 4.5, Gemini 3, Grok 4.1
- **Imagem**: GPT 5 Image, Flux 2 Pro, Riverflow V2

### Uso
```typescript
import { TEXT_MODELS, IMAGE_MODELS, getModelById } from "@/lib/models"
```

## Padrões de Código

### Server vs Client Components
- **Padrão**: Server Components
- **"use client"**: Apenas quando necessário (hooks, eventos, browser APIs)
- **Isolar**: Criar componentes clientes pequenos e específicos

### TypeScript
- **Strict mode**: Habilitado
- **Tipos**: Sempre tipar props, retornos, parâmetros
- **Evitar**: `any` - usar `unknown` se necessário

### Naming Conventions
- Componentes: `PascalCase.tsx`
- Utilitários: `kebab-case.ts`
- Hooks: `usePascalCase.ts`
- Types: `PascalCase.ts`

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
