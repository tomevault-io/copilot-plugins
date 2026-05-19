## cindyia

> Instruções para Claude Code (claude.ai/code) ao trabalhar com este repositório.

# CLAUDE.md

Instruções para Claude Code (claude.ai/code) ao trabalhar com este repositório.

---

## REGRAS OBRIGATÓRIAS

### Comandos Proibidos
- **NUNCA** execute `npm run dev` - o servidor de desenvolvimento é gerenciado pelo usuário
- **NUNCA** execute `npm run lint` ou `npm run build` sem autorização prévia do usuário

### Manutenção da Documentação
- **SEMPRE** atualize os arquivos README.md das pastas ao criar ou deletar arquivos
- Os índices devem refletir a estrutura atual do projeto em tempo real
- Arquivos de índice que referenciam arquivos/pastas modificados devem ser atualizados

### Banco de Dados
- **SEMPRE** use o MCP Server do Supabase (supabase-cindyia) para operações de banco de dados
- **NÃO HÁ** banco de dados rodando localmente - todas as queries devem usar o MCP
- Use as ferramentas `mcp__supabase-cindyia__*` para: listar tabelas, executar SQL, aplicar migrations, etc.

---

## Visão Geral do Projeto

CindyIA é uma plataforma SaaS de agendamento para salões de beleza, focada no mercado brasileiro (UI pt-BR, timezone UTC-3). Usa arquitetura multi-tenant com controle de acesso por roles (super_admin, admin, manager, employee).

---

## Estrutura de Diretórios

```
cindyia/
├── src/                    # Código-fonte React (ver src/README.md)
│   ├── components/         # Componentes React (ver src/components/README.md)
│   ├── hooks/              # Custom hooks (ver src/hooks/README.md)
│   ├── pages/              # Páginas/Rotas (ver src/pages/README.md)
│   ├── integrations/       # Integrações externas
│   ├── lib/                # Utilitários
│   ├── types/              # Type definitions
│   └── utils/              # Utilidades gerais
├── supabase/               # Backend Supabase (ver supabase/README.md)
│   ├── functions/          # Edge Functions (ver supabase/functions/README.md)
│   └── migrations/         # Database migrations
├── public/                 # Assets estáticos (ver public/README.md)
│   └── assets/images/      # Imagens (logo.png)
└── docs/                   # Documentação adicional
```

---

## Navegação Rápida por Arquivo

### Quando precisar modificar...

| Funcionalidade | Arquivo(s) |
|----------------|------------|
| Rotas da aplicação | `src/App.tsx` |
| Layout principal | `src/components/layout/MainLayout.tsx` |
| Autenticação | `src/hooks/useAuth.tsx` |
| Agendamentos | `src/hooks/useAppointments.ts`, `src/pages/Agendamentos.tsx` |
| Clientes | `src/hooks/useClients.ts`, `src/pages/Clientes.tsx` |
| Funcionários | `src/hooks/useEmployees.ts`, `src/pages/Funcionarios.tsx` |
| Serviços | `src/hooks/useServices.ts`, `src/pages/Servicos.tsx` |
| Dashboard | `src/pages/Dashboard.tsx`, `src/components/dashboard/*` |
| Financeiro | `src/hooks/useFinancialEntries.ts`, `src/pages/Financeiro.tsx` |
| Configurações | `src/hooks/useTenantSettings.ts`, `src/pages/Configuracoes.tsx` |
| Componentes UI base | `src/components/ui/*` |
| Edge Functions | `supabase/functions/*/index.ts` |
| Schema do banco | `supabase/migrations/*.sql` |
| Types do Supabase | `src/integrations/supabase/types.ts` |
| Utilitários de data/hora | `src/lib/utils.ts` |
| Logo da aplicação | `public/assets/images/logo.png` |

---

## Comandos Disponíveis

```bash
npm run dev          # Iniciar servidor dev (porta 8080) - NÃO EXECUTAR
npm run build        # Build de produção - REQUER AUTORIZAÇÃO
npm run build:dev    # Build de desenvolvimento
npm run lint         # ESLint - REQUER AUTORIZAÇÃO
npm run preview      # Preview do build
```

---

## Arquitetura

### Stack

| Camada | Tecnologia |
|--------|------------|
| Frontend | React 18 + TypeScript + Vite |
| UI | Tailwind CSS + Shadcn/UI |
| State (Server) | TanStack React Query |
| State (Client) | Zustand |
| Backend | Supabase (PostgreSQL + Auth + Edge Functions) |
| Routing | React Router DOM |

### Padrões

| Padrão | Implementação |
|--------|---------------|
| Path alias | `@/*` → `src/*` |
| Data fetching | React Query via hooks em `src/hooks/` |
| Forms | React Hook Form + Zod |
| Multi-tenancy | Filtro por `tenant_id` em todas as queries |
| Autenticação | Supabase Auth via `useAuth.tsx` |
| Notificações | Sonner toast |
| Tema | next-themes (light/dark) |

### Fluxo de Dados

```
Componente (pages/*)
    → Hook (hooks/use*)
    → Supabase Client
    → PostgreSQL / Edge Functions
```

---

## Timezone

A aplicação opera em **UTC-3 (São Paulo)**.

Helpers disponíveis em `src/lib/utils.ts`:
- `toSaoPauloDateTime()` - Converter para horário SP
- `createSaoPauloDate()` - Criar data em SP
- `getTimeInSaoPaulo()` - Obter hora atual em SP
- `formatTimeInSaoPaulo()` - Formatar hora
- `getDayOfWeekInSaoPaulo()` - Dia da semana
- `getDateInSaoPaulo()` - Data formatada
- `isSameDayInSaoPaulo()` - Comparar datas
- `getTodayInSaoPaulo()` - Data de hoje

---

## TypeScript

Configuração loose (sem strict mode, permite implicit any). Módulos ES2020 com bundler resolution.

---

## Roles e Permissões

| Role | Acesso |
|------|--------|
| super_admin | Todas as rotas + `/admin/*` |
| admin | Rotas do tenant + configurações |
| manager | Rotas operacionais do tenant |
| employee | Acesso limitado (agenda, clientes) |

---

## Índices de Documentação

Para navegação detalhada, consulte os README.md em cada pasta:

- [src/README.md](src/README.md) - Código-fonte principal
- [src/components/README.md](src/components/README.md) - Componentes
- [src/hooks/README.md](src/hooks/README.md) - Custom hooks
- [src/pages/README.md](src/pages/README.md) - Páginas
- [supabase/README.md](supabase/README.md) - Backend
- [supabase/functions/README.md](supabase/functions/README.md) - Edge Functions
- [public/README.md](public/README.md) - Assets estáticos

---
> Source: [Onelord77/cindyia](https://github.com/Onelord77/cindyia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
