## project-overview

> Visão geral do projeto Máquina de Conteúdo - Content Studio


# Máquina de Conteúdo - Content Studio

## Visão Geral

Estúdio de conteúdo alimentado por IA que permite criar, editar e gerenciar posts para redes sociais usando agentes especialistas.

## Tech Stack Principal

- **Framework**: Next.js 16.1.1 (App Router)
- **Linguagem**: TypeScript 5 (strict mode)
- **Estilização**: Tailwind CSS 4 + shadcn/ui (New York style)
- **Autenticação**: Clerk (@clerk/nextjs)
- **Banco de Dados**: Neon PostgreSQL (serverless) + Drizzle ORM
- **State Management**: Zustand
- **LLM Provider**: OpenRouter (multi-modelo)
- **Queue System**: Upstash Redis
- **Ícones**: Lucide React
- **Animações**: Framer Motion + GSAP

## Estrutura de Diretórios

```
src/
├── app/                    # Next.js App Router
│   ├── (app)/             # Rotas protegidas (grupo de rotas)
│   │   ├── dashboard/     # Chat com IA
│   │   ├── library/       # Biblioteca de conteúdo
│   │   ├── calendar/      # Calendário de posts
│   │   ├── sources/       # Fontes de conteúdo
│   │   └── settings/      # Configurações
│   ├── api/               # API Routes
│   │   ├── jobs/          # CRUD de jobs
│   │   ├── workers/       # Worker para processar jobs
│   │   └── webhooks/      # Webhooks (Clerk)
│   ├── sign-in/           # Página de login
│   ├── sign-up/           # Página de registro
│   └── layout.tsx         # Root layout (ClerkProvider)
├── components/
│   ├── app-layout.tsx     # Layout principal (header + nav)
│   ├── auth/              # Componentes de autenticação
│   ├── ui/                # Componentes shadcn/ui
│   └── dashboard/         # Componentes específicos do dashboard
├── db/
│   ├── index.ts           # Cliente Drizzle (Neon serverless)
│   └── schema.ts          # Schema do banco (8 tabelas)
├── lib/
│   ├── queue/             # Sistema de filas (Upstash Redis)
│   ├── models.ts          # Modelos de IA disponíveis
│   └── utils.ts           # Utilitários (cn, etc)
└── hooks/                 # React hooks customizados
```

## Principais Funcionalidades

### 1. Chat com IA
- Interface conversacional para criar conteúdo
- Multi-modelo via OpenRouter (GPT, Claude, Gemini, Grok)
- Histórico de conversas persistido no banco

### 2. Biblioteca de Conteúdo
- Textos, imagens, carrosséis, vídeos, stories
- Status: draft, scheduled, published, archived
- Agendamento de publicação

### 3. Sistema de Filas (Background Jobs)
- Upstash Redis para processamento assíncrono
- Tipos: geração de texto/imagem, criação de carrossel, scraping, publicação agendada
- Workers via API routes (serverless-friendly)

### 4. Base de Conhecimento
- Upload de documentos
- Indexação para RAG (futuro)
- Consulta contextual

### 5. Autenticação
- Clerk com OAuth (Google, GitHub)
- Middleware de proteção de rotas (`src/proxy.ts`)
- Webhook para sincronização de usuários

## Design System

### Cores Principais
- **Background**: `#0a0a0f` (dark)
- **Cards**: Glassmorphism com `bg-[#0a0a0f]/80 backdrop-blur-xl`
- **Borders**: `border-white/10`
- **Primary**: HSL `84 76% 55%` (verde/amarelo)

### Padrões Visuais
- Glassmorphism em cards e header
- Bordas sutis com opacidade
- Glow effects em hover
- Transições suaves

### Componentes UI
- shadcn/ui (New York style)
- Componentes acessíveis (Radix UI)
- Dark mode por padrão

## Variáveis de Ambiente

```env
# Database
DATABASE_URL=postgresql://...

# Clerk Auth
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
CLERK_WEBHOOK_SECRET=whsec_...

# APIs
OPENROUTER_API_KEY=sk-or-...
TAVILY_API_KEY=tvly-...
FIRECRAWL_API_KEY=fc-...
APIFY_API_KEY=apify-...

# Queue System
UPSTASH_REDIS_REST_URL=https://...
UPSTASH_REDIS_REST_TOKEN=...
```

## Comandos Úteis

```bash
# Desenvolvimento
npm run dev

# Build
npm run build

# Lint
npm run lint

# Database
npm run db:generate    # Gerar migration
npm run db:migrate     # Executar migration
npm run db:studio      # UI visual do banco
npm run db:push        # Push schema (sem migration)
```

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
