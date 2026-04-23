## zion-task-copia

> **Zion Tasks** e um sistema de gestao operacional. Ele centraliza tarefas, demandas, timesheet, agentes IA e quadro branco colaborativo.

# Zion Tasks — Guia do Projeto

## Sobre o Projeto
**Zion Tasks** e um sistema de gestao operacional. Ele centraliza tarefas, demandas, timesheet, agentes IA e quadro branco colaborativo.

- **Deploy:** Vercel (auto-deploy via push na `main`)
- **Banco de Dados:** Supabase

---

## Stack Tecnica

| Camada | Tecnologia | Versao |
|--------|-----------|--------|
| **Framework** | React | 19.x |
| **Linguagem** | TypeScript | 5.9.x |
| **Build** | Vite | 8.x |
| **Roteamento** | React Router | 7.x |
| **State Management** | Zustand | 5.x |
| **UI Components** | Radix UI | latest |
| **Icones** | Lucide React | latest |
| **CSS** | Tailwind CSS | 3.4.x |
| **Notificacoes** | Sonner | 2.x |
| **Desenho** | Excalidraw | 0.18.x |
| **IA** | OpenAI SDK | latest |
| **Banco** | Supabase JS | 2.99.x |
| **Testes** | Playwright | 1.58.x |

---

## Comandos Essenciais

```bash
# Instalar dependencias
npm install

# Rodar em desenvolvimento (porta 3333)
npm run dev

# Build de producao
npm run build

# Lint
npm run lint

# Preview do build
npm run preview
```

---

## Variaveis de Ambiente

Copie `.env.example` para `.env` e preencha:

```env
VITE_OPENAI_API_KEY=sk-xxx        # Chave da OpenAI para features de IA
VITE_SUPABASE_URL=https://your-project-ref.supabase.co
VITE_SUPABASE_ANON_KEY=            # Anon key do Supabase
```

**IMPORTANTE:** Nunca commitar o arquivo `.env` com chaves reais.

---

## Estrutura do Projeto

```
src/
├── app/                    # App shell
│   ├── App.tsx             # Layout principal (sidebar + outlet)
│   ├── router.tsx          # Todas as rotas
│   └── providers.tsx       # Context providers
├── features/               # Modulos independentes (feature-based)
│   ├── auth/               # Login (Supabase Auth)
│   ├── kanban/             # Quadro Kanban (tela principal)
│   ├── dashboard/          # Dashboard analitico
│   ├── tasks/              # CRUD de tarefas
│   ├── demands/            # Demandas/requisicoes
│   ├── proposals/          # Propostas ETF e comerciais
│   ├── timesheet/          # Registro de horas
│   ├── team/               # Gestao de time
│   ├── agents/             # Agentes IA + configs
│   ├── transcript/         # Upload e analise de transcricoes
│   ├── drawings/           # Quadro branco (Excalidraw)
│   └── settings/           # Configuracoes (Setores, Projetos, Clientes, Time)
├── services/               # Camada de dados
│   ├── supabase-api.ts     # API Supabase (CRUD real)
│   ├── openai.ts           # Integracao OpenAI
│   ├── api.ts              # API abstrata (fallback)
│   └── mock-api.ts         # Dados mock (desenvolvimento sem Supabase)
├── shared/                 # Codigo compartilhado
│   ├── components/         # ProtectedRoute, etc.
│   ├── lib/                # Cliente Supabase
│   └── types/              # Interfaces TypeScript (Task, Subtask, etc.)
├── assets/                 # Imagens e logos
└── index.css               # Estilos globais + Tailwind
```

---

## Arquitetura e Padroes

### Feature-Based Architecture
Cada modulo em `src/features/` e independente e contem:
- `components/` — Componentes React do modulo
- `store.ts` — Estado local com Zustand
- `index.ts` — Re-exports publicos

### State Management (Zustand)
Cada feature tem sua propria store Zustand. NAO usar estado global compartilhado entre features.

### Multi-Tenant
O sistema e **multi-tenant** — todas as queries filtram por `company_id`. NUNCA esquecer de incluir `company_id` ao criar ou buscar registros.

### Supabase API
Toda comunicacao com o banco passa por `src/services/supabase-api.ts`. NAO criar queries Supabase diretamente nos componentes — sempre usar essa camada de servico.

### Tipos Centralizados
Todos os tipos/interfaces ficam em `src/shared/types/index.ts`. Ao adicionar uma nova entidade, definir o tipo la.

---

## Rotas do App

| Rota | Modulo | Descricao |
|------|--------|-----------|
| `/login` | auth | Tela de login |
| `/` | — | Redireciona para `/kanban` |
| `/kanban` | kanban | Quadro Kanban (tela principal) |
| `/dashboard` | dashboard | Dashboard analitico |
| `/demands` | demands | Lista de demandas |
| `/proposals` | proposals | Lista de propostas |
| `/proposals/new` | proposals | Criar proposta a partir de transcript |
| `/proposals/:id` | proposals | Detalhe de uma proposta |
| `/timesheet` | timesheet | Registro de horas |
| `/team` | team | Gestao de time |
| `/agents` | agents | Agentes IA |
| `/agents/:agentId` | transcript | Analise de transcript por agente |
| `/drawings` | drawings | Galeria de desenhos |
| `/drawings/:id` | drawings | Editor Excalidraw |
| `/settings` | settings | Configuracoes gerais |

Todas as rotas (exceto `/login`) sao protegidas por `ProtectedRoute` (requer autenticacao Supabase).

---

## Tabelas do Banco (Supabase)

| Tabela | Descricao |
|--------|-----------|
| `tasks` | Tarefas do kanban |
| `subtasks` | Subtarefas (vinculadas a tasks) |
| `time_entries` | Registros de tempo |
| `sectors` | Setores da empresa |
| `projects` | Projetos (vinculados a setores) |
| `clients` | Clientes |
| `users` | Usuarios do sistema |
| `proposals` | Propostas ETF e comerciais |
| `drawings` | Desenhos do quadro branco |

**REGRA:** Todas as tabelas usam `company_id` para isolamento multi-tenant. RLS (Row Level Security) esta ativo.

---

## Tipos Principais

```typescript
// Status de tarefa
type TaskStatus = 'todo' | 'in_progress' | 'review' | 'done'

// Prioridade
type Priority = 'low' | 'medium' | 'high' | 'urgent'

// Tipo de demanda
type DemandType = 'bug' | 'feature' | 'improvement' | 'update' | 'tech_debt'

// Status de demanda
type DemandStatus = 'triagem' | 'aprovado' | 'rejeitado' | 'aguardando_info'

// Status de projeto
type ProjectStatus = 'active' | 'paused' | 'archived'

// Tipo de proposta
type ProposalType = 'etf' | 'proposta_comercial'

// Status de proposta
type ProposalStatus = 'rascunho' | 'revisao_interna' | 'enviado' | 'aprovado' | 'rejeitado' | 'ajuste_solicitado'
```

---

## Regras de Desenvolvimento

### Git Flow
- **Branch principal:** `main`
- **Para contribuicoes:** Criar branch a partir de `main` (ex: `feature/nome-da-feature`, `fix/descricao-do-bug`)
- **Pull Requests:** Obrigatorios para merge na `main`
- **Commits:** Mensagens claras em portugues ou ingles, prefixadas (`feat:`, `fix:`, `refactor:`, `chore:`)

### Antes de Commitar
1. Rodar `npm run build` — garantir que compila sem erros
2. Rodar `npm run lint` — corrigir warnings/erros de lint
3. Testar a funcionalidade alterada no navegador

### Boas Praticas
- Manter componentes pequenos e focados
- Usar TypeScript strict — NAO usar `any` desnecessariamente
- Seguir a arquitetura feature-based existente
- Manter a camada de servico (`supabase-api.ts`) como unica interface com o banco
- Labels em portugues brasileiro na UI
- NUNCA commitar `.env` com credenciais reais

---

## Idioma
Toda comunicacao, commits, e labels do sistema sao em **portugues brasileiro**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ziontrafficservicos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
