## corporis-finance

> > Este arquivo é a fonte de verdade do projeto. O Claude Code lê este arquivo automaticamente em toda sessão. **Toda decisão técnica importante mora aqui.** Quando algo do projeto evoluir, atualize este arquivo no mesmo PR.

# CLAUDE.md — Corporis Finance

> Este arquivo é a fonte de verdade do projeto. O Claude Code lê este arquivo automaticamente em toda sessão. **Toda decisão técnica importante mora aqui.** Quando algo do projeto evoluir, atualize este arquivo no mesmo PR.

---

## 1. O que é este projeto

**Corporis Finance** é uma plataforma web de gestão financeira empresarial focada em **DFC (Demonstrativo de Fluxo de Caixa)**, construída inicialmente para a clínica **Corporis Fisioterapia & Pilates** (Xanxerê/SC).

**MVP single-tenant** (uma única clínica, um único usuário gestor). A arquitetura, no entanto, é desenhada para evoluir para multi-tenant SaaS no futuro — toda tabela tem `organization_id` desde o dia 1.

### Princípios de produto

1. **Caixa é rei.** Todo o sistema opera em regime de caixa. Competência fica para uma futura versão com DRE.
2. **O DFC é o coração.** Toda feature precisa responder: "isso melhora a clareza ou velocidade de produzir o DFC?"
3. **Dados realistas, não Lorem ipsum.** Em dev, seed com dados verossímeis de uma clínica de fisio+pilates.
4. **Tom acolhedor.** "Aluna", "atendimento", "movimentação". Nunca "paciente" ou "transação financeira".
5. **Não reinventar a contabilidade.** O plano de contas Corporis é a referência. Não criar grupos novos sem necessidade.

### Princípios técnicos

1. **Tipos primeiro.** TypeScript estrito. Schema do banco define tipos via Supabase codegen.
2. **Server Components por padrão.** Client Components apenas quando precisa de estado, eventos ou hooks.
3. **RLS em todas as tabelas, sempre.** Mesmo em single-tenant. Nunca commitar tabela sem policy.
4. **Erros falam português.** Mensagens de UI são em pt-BR. Logs e código em inglês.
5. **Sem feature flag gambiarra.** Se vai construir, constrói direito. Se ainda não vai, não constrói.

---

## 1.1 Como usar a pasta `/design`

> **Regra absoluta:** toda tela implementada deve ser fiel ao mockup aprovado em `/design/mockups/`. O Claude Code não toma decisões visuais autônomas — qualquer dúvida de layout ou componente, a resposta está no HTML de referência.

### Estrutura da pasta

```
/design
  /mockups
    01-dashboard.html          ← mockup interativo completo (fonte de verdade visual)
    01-dashboard.png           ← screenshot para referência rápida
    02-lancamentos-listagem.html
    02-lancamentos-listagem.png
    03-lancamentos-modal.html
    03-lancamentos-modal.png
    04-importacao-upload.html
    04-importacao-upload.png
    05-importacao-conciliacao.html
    05-importacao-conciliacao.png
    06-contas-listagem.html
    06-contas-listagem.png
    07-contas-cadastro.html
    07-contas-cadastro.png
    08-dfc.html
    08-dfc.png
    09-orcado-realizado.html
    09-orcado-realizado.png
    10-orcamento-editor.html
    10-orcamento-editor.png
    11-projecao.html
    11-projecao.png
    12-consultor-ia.html
    12-consultor-ia.png
    13-plano-de-contas.html
    13-plano-de-contas.png
    14-importacoes-historico.html
    14-importacoes-historico.png
    15-configuracoes.html
    15-configuracoes.png
    16-login.html
    16-login.png
    17-onboarding.html
    17-onboarding.png
  DLS.md                       ← Design Language System completo (tokens, componentes, regras)
  SCREENS.md                   ← índice de todas as telas com status de aprovação
```

### Como o Claude Code deve ler os mockups

Ao começar a implementar qualquer tela, **leia o HTML de referência antes de escrever uma linha de código**. Extraia:

1. **Estrutura de layout** — grid, flex, proporções de colunas, larguras fixas
2. **Hierarquia de componentes** — quais componentes existem e como se encaixam
3. **Tokens aplicados** — cores, espaçamentos, tipografia, raios de borda usados
4. **Dados exibidos** — quais campos existem, formatos de data/moeda, labels
5. **Estados visíveis** — o mockup mostra um estado específico; inferir os outros (vazio, erro, loading) a partir do DLS.md
6. **Interações indicadas** — comentários no HTML descrevem hover, modais, transições

### Hierarquia de decisão visual

```
1. Mockup HTML aprovado em /design/mockups/   → fonte de verdade
2. DLS.md em /design/                         → tokens e regras
3. Componentes shadcn/ui customizados          → base de implementação
4. Intuição do Claude Code                     → NUNCA — sempre consultar 1 ou 2
```

### Quando o mockup e o DLS conflitarem

O mockup prevalece — ele foi produzido a partir do DLS e representa a decisão final. Se o conflito for genuíno (ex: mockup usa uma cor fora do DLS), **parar e perguntar** antes de resolver por conta própria.

### SCREENS.md — índice de aprovação

Antes de implementar qualquer tela, verifique o status em `design/SCREENS.md`. Só implemente telas com status `✅ aprovado`. Se estiver como `🔄 revisão`, aguarde aprovação antes de começar.

---

## 2. Stack

| Camada | Tecnologia | Versão | Por quê |
|---|---|---|---|
| Framework | **Next.js** (App Router) | 15.x | RSC, Server Actions, ótima DX |
| Linguagem | **TypeScript** | 5.x estrito | Segurança de tipos no domínio financeiro é não-negociável |
| UI | **React** | 19.x | — |
| Estilização | **Tailwind CSS** | 4.x | Match com o sistema de tokens do DLS |
| Componentes | **shadcn/ui** | latest | Base não-opinativa, customizável para o DLS Corporis |
| Ícones | **lucide-react** | latest | Stroke 1.5, alinhado com o DLS |
| Gráficos | **Recharts** | latest | Composição declarativa, fácil de tematizar |
| Formulários | **react-hook-form + zod** | latest | Validação forte + DX excelente |
| Tabelas | **TanStack Table** | v8 | Tabelas densas com filtros, sorting, virtualização |
| Datas | **date-fns** + **date-fns-tz** | latest | Timezone-aware, locale pt-BR |
| Backend/DB | **Supabase** (Postgres + Auth + Storage + Edge Functions) | latest | DB + autenticação + storage com RLS pronto |
| ORM/Acesso | **Supabase JS client** (server e browser) + SQL puro quando necessário | — | Mantém leve, sem ORM pesado |
| IA | **Anthropic SDK** (Claude Sonnet) | latest | Categorização e parsing — pt-BR forte |
| Parsing OFX | **node-ofx-parser** ou parser próprio | — | OFX brasileiro tem peculiaridades |
| Parsing PDF | Claude Vision direto (PDF → texto estruturado) | — | Faturas variam muito entre bancos |
| Parsing CSV | **papaparse** | latest | Padrão da indústria |
| Testes | **Vitest** + **Playwright** | latest | Unit + E2E |
| Linter | **Biome** | latest | Lint + format em um só, rápido |
| Gerenciador | **pnpm** | latest | Performance e workspaces |
| Hospedagem | Local em dev, **Vercel** + **Supabase Cloud** em prod (decisão futura) | — | — |

### Por que NÃO usamos
- **Prisma/Drizzle:** o Supabase client com tipos gerados resolve. Menos camadas.
- **Redux/Zustand:** Server Components + URL state + React state local resolve 95% dos casos.
- **NextAuth:** Supabase Auth já está incluído.
- **Axios/SWR/Tanstack Query:** Server Actions + revalidatePath cobrem o MVP.

---

## 3. Estrutura de pastas

```
corporis-finance/
├─ .claude/                          # Configurações do Claude Code
│  ├─ commands/                      # Slash commands customizados
│  │  ├─ new-feature.md
│  │  ├─ new-migration.md
│  │  └─ review-rls.md
│  └─ settings.json
├─ .vscode/
├─ docs/                             # Documentação viva
│  ├─ DESIGN_SYSTEM.md               # Tokens, componentes, exemplos
│  ├─ DATA_MODEL.md                  # ER + decisões de modelagem
│  ├─ DFC_LOGIC.md                   # Como o DFC é calculado
│  ├─ AI_FEATURES.md                 # Prompts e fluxos de IA
│  └─ mockups/                       # HTMLs gerados pelo Prompt 1
├─ public/
│  ├─ logo/                          # Variações do logo
│  └─ fonts/                         # Olicy (se disponível) + fallback
├─ src/
│  ├─ app/                           # Next.js App Router
│  │  ├─ (auth)/                     # Rotas públicas
│  │  │  ├─ login/page.tsx
│  │  │  └─ layout.tsx
│  │  ├─ (app)/                      # Rotas autenticadas
│  │  │  ├─ dashboard/page.tsx
│  │  │  ├─ lancamentos/
│  │  │  ├─ contas/
│  │  │  ├─ dfc/
│  │  │  ├─ orcamento/
│  │  │  ├─ projecao/
│  │  │  ├─ consultor/
│  │  │  ├─ plano-de-contas/
│  │  │  ├─ importacoes/
│  │  │  ├─ configuracoes/
│  │  │  └─ layout.tsx               # Sidebar + conteúdo principal
│  │  ├─ api/
│  │  │  ├─ import/parse-ofx/route.ts
│  │  │  ├─ import/parse-csv/route.ts
│  │  │  ├─ import/parse-pdf/route.ts   # usa Claude Vision
│  │  │  ├─ ai/categorize/route.ts
│  │  │  └─ ai/chat/route.ts
│  │  ├─ onboarding/
│  │  ├─ layout.tsx
│  │  └─ globals.css
│  ├─ components/
│  │  ├─ ui/                         # shadcn/ui customizado para Corporis
│  │  ├─ layout/                     # Sidebar, AppShell
│  │  ├─ charts/                     # Wrappers do Recharts com tema Corporis
│  │  ├─ forms/                      # Form components compostos
│  │  ├─ tables/                     # DataTable, FilterBar
│  │  ├─ money/                      # MoneyInput, MoneyDisplay, MoneyDelta
│  │  ├─ category/                   # CategoryPicker, CategoryBadge, CategoryTree
│  │  ├─ account/                    # AccountCard, AccountPicker
│  │  └─ ai/                         # AISuggestion, AIBadge, ChatMessage
│  ├─ lib/
│  │  ├─ supabase/
│  │  │  ├─ client.ts                # Browser client
│  │  │  ├─ server.ts                # Server client (cookies)
│  │  │  ├─ admin.ts                 # Service role (apenas em scripts/edge)
│  │  │  └─ types.ts                 # Tipos gerados via supabase gen types
│  │  ├─ dfc/
│  │  │  ├─ calculate.ts             # Lógica do DFC
│  │  │  ├─ vertical-analysis.ts     # AV%
│  │  │  ├─ projection.ts            # Projeção de caixa
│  │  │  └─ budget-vs-actual.ts
│  │  ├─ import/
│  │  │  ├─ ofx-parser.ts
│  │  │  ├─ csv-parser.ts
│  │  │  ├─ pdf-parser.ts            # invoca Claude Vision
│  │  │  └─ dedup.ts                 # detecção de duplicatas
│  │  ├─ ai/
│  │  │  ├─ client.ts                # Anthropic SDK wrapper
│  │  │  ├─ prompts.ts               # Prompts versionados
│  │  │  └─ categorize.ts
│  │  ├─ money.ts                    # formatBRL, parseBRL, Decimal helpers
│  │  ├─ date.ts                     # helpers de data com TZ America/Sao_Paulo
│  │  └─ utils.ts                    # cn(), debounce, etc
│  ├─ hooks/                         # Hooks custom (usar com moderação)
│  ├─ actions/                       # Server Actions
│  │  ├─ transactions.ts
│  │  ├─ accounts.ts
│  │  ├─ budget.ts
│  │  ├─ chart-of-accounts.ts
│  │  └─ imports.ts
│  ├─ types/                         # Tipos de domínio (Transaction, Account, ...)
│  └─ styles/
├─ supabase/                         # Supabase CLI workspace
│  ├─ migrations/                    # SQL migrations versionadas
│  ├─ seed.sql                       # Seed de dev (plano de contas + dados Corporis)
│  ├─ functions/                     # Edge Functions (futuro)
│  └─ config.toml
├─ scripts/
│  ├─ seed-dev.ts                    # Popula dados de exemplo
│  └─ generate-types.sh              # Roda supabase gen types
├─ tests/
│  ├─ unit/
│  └─ e2e/
├─ .env.local.example
├─ .env.local                        # NUNCA commitar
├─ biome.json
├─ next.config.ts
├─ package.json
├─ pnpm-lock.yaml
├─ playwright.config.ts
├─ tailwind.config.ts
├─ tsconfig.json
├─ vitest.config.ts
└─ CLAUDE.md                         # Este arquivo
```

---

## 4. MCPs e ferramentas do Claude Code

Adicione estes MCPs ao seu Claude Code (`.claude/settings.json` ou `claude mcp add`):

| MCP | Por quê | Como adicionar |
|---|---|---|
| **Supabase MCP** (oficial) | Inspecionar schema, rodar queries, listar policies sem sair do terminal | `claude mcp add supabase` e configurar com Project URL + access token |
| **Postgres MCP** | Para queries diretas no DB local em dev | Conforme docs do MCP do PG |
| **Filesystem MCP** | Já vem por padrão | — |
| **Playwright MCP** | Inspecionar telas durante dev, debugar visualmente | `claude mcp add playwright` |
| **Context7 / docs MCP** (opcional) | Buscar docs do Next.js, Tailwind, Supabase de forma consistente | Conforme docs |

> **Configuração de cada MCP fica fora deste arquivo** porque depende de secrets. Mantenha um `docs/SETUP_MCPS.md` com instruções (sem segredos).

---

## 5. Schema do banco (resumo — detalhes em `supabase/migrations/`)

> Convenções: `snake_case`, IDs `uuid` com `gen_random_uuid()`, todo timestamp em `timestamptz`, todo valor monetário em `numeric(14,2)`, todas as tabelas com `organization_id`, `created_at`, `updated_at` e RLS.

### Tabelas principais

**`organizations`**
```sql
id uuid PK
name text NOT NULL
slug text UNIQUE
locale text DEFAULT 'pt-BR'
timezone text DEFAULT 'America/Sao_Paulo'
currency text DEFAULT 'BRL'
created_at, updated_at
```

**`profiles`** (extends auth.users)
```sql
id uuid PK references auth.users(id)
organization_id uuid references organizations(id)
full_name text
role text CHECK (role IN ('owner', 'admin', 'viewer')) DEFAULT 'owner'
created_at, updated_at
```

**`accounts`** (contas e cartões)
```sql
id uuid PK
organization_id uuid
name text NOT NULL                     -- "Itaú PJ"
type text CHECK (type IN ('checking', 'savings', 'cash', 'credit_card'))
bank_name text                         -- "Itaú", "Nubank"
color text                             -- hex da paleta
opening_balance numeric(14,2) DEFAULT 0
opening_balance_date date
-- campos específicos de cartão de crédito:
credit_limit numeric(14,2)
closing_day int                        -- dia do fechamento da fatura
due_day int                            -- dia do vencimento da fatura
default_payment_account_id uuid references accounts(id)
is_active boolean DEFAULT true
created_at, updated_at
```

**`chart_of_accounts`** (plano de contas — hierárquico)
```sql
id uuid PK
organization_id uuid
parent_id uuid references chart_of_accounts(id) ON DELETE RESTRICT
code text NOT NULL                     -- "4.01.01"
name text NOT NULL                     -- "Salário"
nature text CHECK (nature IN ('income', 'expense', 'transfer', 'calculated'))
dfc_group text CHECK (dfc_group IN ('operational', 'non_operational', 'investment', 'financing'))
cost_classification text CHECK (cost_classification IN ('fixed', 'variable', NULL))
display_order int
is_active boolean DEFAULT true
created_at, updated_at
UNIQUE(organization_id, code)
```

**`transactions`** (lançamentos)
```sql
id uuid PK
organization_id uuid
account_id uuid references accounts(id)
category_id uuid references chart_of_accounts(id)
counter_account_id uuid references accounts(id)  -- transferências
type text CHECK (type IN ('income', 'expense', 'transfer'))
amount numeric(14,2) NOT NULL CHECK (amount > 0)  -- sempre positivo, sinal vem do type
description text NOT NULL
-- Datas: separação crítica para cartão
event_date date NOT NULL               -- data do fato (compra, recebimento)
cash_date date NOT NULL                -- data do efeito caixa (para o DFC)
status text CHECK (status IN ('pending', 'cleared')) DEFAULT 'cleared'
-- Para lançamentos de fatura de cartão:
credit_card_invoice_id uuid references credit_card_invoices(id)
-- Origem
source text CHECK (source IN ('manual', 'import_ofx', 'import_csv', 'import_pdf', 'recurring')) DEFAULT 'manual'
import_id uuid references imports(id)
external_id text                       -- hash do banco para dedup
-- IA
ai_categorized boolean DEFAULT false
ai_confidence numeric(3,2)             -- 0.00 a 1.00
notes text
created_at, updated_at
```

**`credit_card_invoices`** (faturas de cartão)
```sql
id uuid PK
organization_id uuid
account_id uuid references accounts(id)  -- o cartão
closing_date date NOT NULL
due_date date NOT NULL
total_amount numeric(14,2)
paid_amount numeric(14,2) DEFAULT 0
status text CHECK (status IN ('open', 'closed', 'paid', 'partially_paid'))
payment_transaction_id uuid references transactions(id)  -- a transação de pagamento da fatura
created_at, updated_at
```

**`attachments`**
```sql
id uuid PK
organization_id uuid
transaction_id uuid references transactions(id) ON DELETE CASCADE
storage_path text NOT NULL             -- caminho no Supabase Storage
filename text
mime_type text
size_bytes int
created_at
```

**`imports`** (histórico de importações)
```sql
id uuid PK
organization_id uuid
account_id uuid
import_type text CHECK (import_type IN ('ofx', 'csv', 'pdf_invoice'))
filename text
status text CHECK (status IN ('pending', 'parsing', 'reviewing', 'completed', 'failed'))
total_rows int
imported_rows int
duplicates_found int
raw_content text                       -- para debug (truncar se muito grande)
error_message text
created_at, completed_at
```

**`budgets`** (orçamento anual)
```sql
id uuid PK
organization_id uuid
year int NOT NULL
version int NOT NULL DEFAULT 1
name text                              -- "v1 — Janeiro", "v2 — Revisão Junho"
is_active boolean DEFAULT true         -- só uma versão ativa por ano
created_at, updated_at
UNIQUE(organization_id, year, version)
```

**`budget_lines`** (linhas do orçamento)
```sql
id uuid PK
organization_id uuid
budget_id uuid references budgets(id) ON DELETE CASCADE
category_id uuid references chart_of_accounts(id)
month int CHECK (month BETWEEN 1 AND 12)
amount numeric(14,2) DEFAULT 0
UNIQUE(budget_id, category_id, month)
```

**`ai_conversations` + `ai_messages`** (chat do consultor)
```sql
ai_conversations: id, organization_id, title, created_at, updated_at
ai_messages: id, conversation_id, role ('user'|'assistant'|'system'), content, tool_calls jsonb, created_at
```

**`ai_categorization_rules`** (regras aprendidas para sugestões)
```sql
id uuid PK
organization_id uuid
description_pattern text               -- texto que casou
category_id uuid references chart_of_accounts(id)
match_count int DEFAULT 1
last_used_at timestamptz
```

### Índices essenciais
```sql
CREATE INDEX ON transactions(organization_id, cash_date DESC);
CREATE INDEX ON transactions(organization_id, account_id, cash_date DESC);
CREATE INDEX ON transactions(organization_id, category_id);
CREATE INDEX ON transactions(organization_id, credit_card_invoice_id);
CREATE INDEX ON chart_of_accounts(organization_id, parent_id, display_order);
CREATE INDEX ON budget_lines(budget_id, category_id, month);
```

---

## 6. RLS — Política universal

**Toda tabela** com `organization_id` segue este padrão. Migration template em `supabase/migrations/_template_rls.sql`.

```sql
ALTER TABLE <tabela> ENABLE ROW LEVEL SECURITY;

CREATE POLICY "<tabela>_org_isolation" ON <tabela>
  FOR ALL
  TO authenticated
  USING (organization_id = (SELECT organization_id FROM profiles WHERE id = auth.uid()))
  WITH CHECK (organization_id = (SELECT organization_id FROM profiles WHERE id = auth.uid()));
```

**Regras:**
- Nenhuma tabela vai para produção sem RLS habilitada
- Service role só é usado em scripts e Edge Functions, nunca em código de client
- Em PR review, todo SQL novo precisa de checklist de RLS

---

## 7. Cálculo do DFC — lógica central

Documentação completa em `docs/DFC_LOGIC.md`. Resumo:

### Regime: caixa puro
Toda transação afeta o DFC na sua `cash_date`, não na `event_date`.

### Cartão de crédito
- Compra no cartão em 03/nov: cria transação com `event_date=03/nov`, `cash_date=10/dez` (vencimento), `credit_card_invoice_id` apontando para a fatura
- O DFC vê a despesa em **dezembro** (mês do pagamento da fatura)
- Cada compra mantém sua categoria individual (Materiais Fisio, Sistemas, etc) — assim a análise por categoria continua precisa, só que com lag de caixa

### Transferências
- Tipo `transfer` tem `account_id` (origem) e `counter_account_id` (destino)
- **NÃO** aparece em nenhuma linha do DFC, só impacta saldo de cada conta
- Aparece no extrato de cada conta com sinal correspondente

### Estrutura do DFC (clássico, método direto)
```
(+) ENTRADAS OPERACIONAIS                ← Grupo 1
(−) IMPOSTOS                              ← Grupo 2
(−) CUSTOS                                ← Grupo 3
= LUCRO BRUTO

(−) DESPESAS OPERACIONAIS                 ← Grupo 4 (4.1 + 4.2 + 4.3)
= LUCRO LÍQUIDO

= FLUXO DE CAIXA OPERACIONAL              ← Linhas acima

(+/−) REC/DESP FINANCEIRA                 ← Grupo 5
(+/−) RESULTADO NÃO OPERACIONAL           ← Grupo 6
= FLUXO DE CAIXA NÃO OPERACIONAL          ← Grupos 5 + 6

(−) INVESTIMENTOS                         ← Grupo 7
= FLUXO DE CAIXA DOS INVESTIMENTOS

(+/−) FINANCEIRO                          ← Grupo 8
= FLUXO DE CAIXA FINANCEIRO

= FLUXO DE CAIXA LIVRE                    ← Soma de tudo
= SALDO FINAL REAL                        ← Saldo inicial do mês + Fluxo Livre
```

### Análise Vertical (AV%)
Cada linha = `valor_da_linha / total_entradas_do_periodo × 100`. Total de entradas = soma do Grupo 1 (Receita Bruta).

### Projeção
Combinação ponderada de:
1. Lançamentos com `status='pending'` e `cash_date` no futuro (peso 1.0)
2. Recorrências detectadas (futuro — V2)
3. Médias móveis dos últimos 3-6 meses por categoria (peso decrescente quanto mais longe no futuro)

Cenários:
- Conservador: entradas −20%, saídas +10%
- Realista: valores médios
- Otimista: entradas +10%, saídas −5%

---

## 8. Features de IA — uso de Claude

Documentação completa em `docs/AI_FEATURES.md`. Resumo:

### Feature A — Categorização automática (importação)
**Quando:** após parse de OFX/CSV/PDF, antes do usuário revisar
**Modelo:** Claude Sonnet 4.5 (mais novo) via Anthropic SDK
**Input:** lista de transações sem categoria + plano de contas completo + últimos 200 lançamentos do usuário (como exemplos few-shot)
**Output:** JSON estruturado com `category_id` e `confidence` (0-1) por transação
**UI:** badge "IA sugeriu" com nível de confiança, usuário confirma/edita

### Feature C — Parser de PDF de fatura
**Quando:** upload de PDF de fatura de cartão
**Modelo:** Claude com input multimodal (PDF como imagem)
**Input:** PDF da fatura
**Output:** JSON estruturado: `{ closing_date, due_date, total, transactions: [{ date, description, amount, installment_info? }] }`
**Pós-processamento:** envia para Feature A para categorizar

### Princípios para IA
- Prompts versionados em `src/lib/ai/prompts.ts` (mudança de prompt = mudança no histórico)
- Sempre validar output da IA contra zod schema antes de usar
- Logs de uso (tokens in/out) por chamada
- Nunca usar IA para gerar valores monetários sem confirmação humana
- Rate limit no backend para evitar custos surpresa

---

## 9. Padrões de código

### TypeScript
- `strict: true`, `noUncheckedIndexedAccess: true`
- Nada de `any`. `unknown` quando necessário, com narrowing
- Tipos de domínio em `src/types/`, **não** importar tipos do Supabase direto em componentes — sempre passar por uma camada de "model" em `src/lib/`

### Server vs Client Components
- Default: **Server Component**
- Client Component só quando: precisa de `useState`, `useEffect`, event handlers, browser APIs
- `"use client"` sempre no menor componente possível
- Server Actions para mutações (formulários, deletar, etc), com revalidação via `revalidatePath`

### Formulários
- `react-hook-form` + `zod` resolver
- Schema zod compartilhado entre client (validação no form) e server action (validação na borda)
- Mensagens de erro em pt-BR

### Dinheiro
- **NUNCA** usar `number` para valores monetários em lógica de cálculo
- Usar `string` (do Postgres `numeric`) e `Decimal.js` (ou similar) para somas
- Helpers em `src/lib/money.ts`: `formatBRL(value): string`, `parseBRL(input): string`, `sumMoney(values): string`
- Display: sempre `R$ X.XXX,XX` com `font-variant-numeric: tabular-nums`

### Datas
- Sempre armazenar como `date` ou `timestamptz` no banco
- TZ da organização: `America/Sao_Paulo`
- Helpers em `src/lib/date.ts` para conversão e formatação
- Cuidado especial em projeção — usar `startOfDay`/`endOfDay` da `date-fns-tz`

### Nomes
- Componentes: `PascalCase`
- Arquivos de componente: `kebab-case.tsx` (ex: `transaction-row.tsx`)
- Server Actions: `verb-noun.ts` (ex: `create-transaction.ts`)
- Variáveis e funções: `camelCase`
- Constantes globais: `SCREAMING_SNAKE_CASE`
- Tabelas e colunas SQL: `snake_case`

### Comentários
- Em português é OK para regras de negócio complexas (DFC, projeção)
- Código geral em inglês
- Não comentar o óbvio
- Use comentários para "**por quê**", não "**o quê**"

---

## 10. Comandos essenciais

```bash
# Setup inicial
pnpm install
cp .env.local.example .env.local       # preencher chaves
pnpm supabase start                     # sobe Postgres + Auth + Storage local
pnpm db:reset                           # aplica migrations + seed
pnpm db:types                           # gera tipos TS a partir do schema

# Dev
pnpm dev                                # Next.js em :3000
pnpm supabase status                    # checar serviços locais

# Migrations
pnpm supabase migration new <nome>      # cria nova migration
pnpm db:reset                           # reaplica tudo (CUIDADO: apaga dados locais)

# Qualidade
pnpm lint
pnpm format
pnpm typecheck
pnpm test                               # unit
pnpm test:e2e                           # Playwright

# Build
pnpm build
pnpm start
```

### Scripts no `package.json`
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "biome check .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:e2e": "playwright test",
    "db:reset": "supabase db reset",
    "db:types": "supabase gen types typescript --local > src/lib/supabase/types.ts",
    "db:seed": "tsx scripts/seed-dev.ts"
  }
}
```

---

## 11. Variáveis de ambiente

`.env.local.example`:
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=<local-anon-key>
SUPABASE_SERVICE_ROLE_KEY=<local-service-role>   # APENAS server-side

# IA
ANTHROPIC_API_KEY=sk-ant-...

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Regra:** nada que comece com `NEXT_PUBLIC_` pode conter segredo. Service role NUNCA aparece em código de client.

---

## 12. Critério de pronto (Definition of Done) por feature

Uma feature só está pronta quando:

- [ ] Tem migration aplicada com RLS
- [ ] Server Action ou rota API com validação zod
- [ ] UI segue o DLS Corporis (revisado contra `docs/DESIGN_SYSTEM.md`)
- [ ] Estados: loading, vazio, erro
- [ ] Mensagens de erro em pt-BR e acolhedoras
- [ ] Pelo menos 1 teste unitário se tem lógica não-trivial
- [ ] Funciona com 80+ lançamentos sem lag perceptível
- [ ] Acessibilidade básica: foco visível, labels, contraste AA
- [ ] CLAUDE.md atualizado se houve decisão arquitetural

---

## 13. Roadmap do MVP (ordem sugerida de construção)

> Não pule etapas. Cada uma depende da anterior.

1. **Fundação**
   - Setup do projeto (Next + Supabase + Biome + shadcn)
   - Tema Tailwind com tokens Corporis
   - Componentes base: Button, Input, Card, Badge, Money, AppShell
   - Auth (Supabase Auth) + onboarding (telas 16, 17)

2. **Domínio mínimo**
   - Migration: organizations, profiles, accounts, chart_of_accounts (com seed Corporis)
   - Tela 13 — Plano de Contas (visualizar e editar)
   - Tela 06 — Contas e Cartões (CRUD)

3. **Lançamentos manuais**
   - Migration: transactions, attachments
   - Tela 02 — Listagem com filtros e tabela
   - Tela 03 — Modal de novo/editar
   - Lógica de transferência

4. **Dashboard**
   - Tela 01 — Dashboard com KPIs e gráficos básicos

5. **DFC**
   - `lib/dfc/calculate.ts` com testes pesados
   - Tela 08 — DFC com expansão por grupo, AV%, comparação

6. **Importação manual + IA**
   - Migration: imports, credit_card_invoices
   - Parser OFX/CSV
   - Tela 04 — Upload
   - Tela 05 — Conciliação com IA (categorização)
   - Parser PDF via Claude Vision
   - Lógica de cartão (event_date vs cash_date)

7. **Orçamento**
   - Migration: budgets, budget_lines
   - Tela 10 — Editor de orçamento
   - Tela 09 — Orçado x Realizado

8. **Projeção**
   - `lib/dfc/projection.ts`
   - Tela 11 — Projeção de caixa

9. **Consultor IA**
   - Tela 12 — Chat com Claude e tools (consultar transações, etc)

10. **Polish e configurações**
    - Tela 14 — Importações histórico
    - Tela 15 — Configurações
    - Testes E2E principais
    - Decisão de hospedagem

---

## 14. Como pedir ajuda ao Claude Code

Ao iniciar uma sessão para construir algo, comece com:

> *"Lendo o CLAUDE.md. Vou trabalhar em [feature X]. Antes de começar, me confirme: [decisão Y do CLAUDE.md] continua valendo?"*

Ao terminar uma feature significativa:

> *"Atualize o CLAUDE.md com a decisão de [Z]. Adicione um teste se faltou. Rode lint e typecheck."*

Para schema novo:

> *"Crie uma migration nova chamada [nome]. Inclua RLS no template padrão. Atualize a seção 5 do CLAUDE.md com a nova tabela."*

---

## 15. Anti-padrões — não faça

- ❌ Tabela sem RLS
- ❌ Valores monetários em `number` JavaScript
- ❌ `any` em TypeScript
- ❌ Cor fora da paleta Corporis (azul, roxo, preto puro, branco puro)
- ❌ Componente client desnecessário (`"use client"` no topo de tudo)
- ❌ Lógica de DFC dentro de componente — sempre em `lib/dfc/`
- ❌ Importar tipos do Supabase direto em UI — passar por camada de model
- ❌ Texto fixo em inglês em UI ("Save", "Cancel") — sempre pt-BR
- ❌ "Cliente" ou "paciente" — usar "aluna" ou "atendimento"
- ❌ Usar IA para somar dinheiro
- ❌ Commit com `console.log` de produção
- ❌ Adicionar dependência sem documentar aqui o porquê

---

## 16. Guia de implementação por etapas

> **Como usar:** no início de cada sessão de desenvolvimento, informe ao Claude Code em qual etapa você está. Ele vai saber exatamente o que construir, em que ordem, e quando parar. Nunca avance para a próxima etapa sem passar pelo critério de saída da atual — isso evita retrabalho e dependências quebradas.
>
> **Convenção de status:**
> - `[ ]` — não iniciado
> - `[~]` — em andamento
> - `[x]` — concluído e validado

---

### ETAPA 1 — Fundação do projeto
*Objetivo: ambiente rodando, tema visual aplicado, componentes base prontos. Nenhuma lógica de negócio ainda.*

**O que construir, nesta ordem:**

1. `[x]` Criar projeto Next.js 15 com TypeScript estrito (`pnpm create next-app`)
2. `[x]` Instalar e configurar Tailwind 4 + shadcn/ui
3. `[x]` Aplicar tokens do DLS Corporis no `tailwind.config.ts` (cores, fontes, sombras, raios)
4. `[x]` Importar fontes Fraunces + Ubuntu do Google Fonts no `layout.tsx` raiz
5. `[x]` Configurar Biome (lint + format) e `tsconfig.json` estrito
6. `[x]` Iniciar Supabase local (`supabase init` + `supabase start`) e preencher `.env.local` — Docker Desktop instalado (Apple Silicon, v29.4.3). `pnpm supabase start` OK; API :54321 (REST 200), Studio :54323, DB :54322. `.env.local` preenchido com anon/service keys reais (imgproxy/pooler ficam parados — opcionais, ok)
7. `[x]` Construir componentes base (em `src/components/ui/`, sobre shadcn):
   - `[x]` `Button` (primary, secondary, ghost — cores Corporis)
   - `[x]` `Input` com foco laranja
   - `[x]` `Card` com border e shadow quentes
   - `[x]` `Badge` / `CategoryBadge` com mapeamento de grupos do plano de contas
   - `[x]` `MoneyDisplay` (formata R$ com tabular-nums, cor por sinal)
   - `[x]` `MoneyInput` (input mascarado para valores BRL)
8. `[x]` Construir `AppShell`: Sidebar (240px, 10 itens na ordem do CLAUDE.md) + área de conteúdo sem cabeçalho global. Decisão posterior: Topbar removido do app; sino de notificações mantido na sidebar e implementado como popover in-app com notificações persistidas
9. `[x]` Criar rotas vazias para cada módulo em `src/app/(app)/` — layout com AppShell + `<h1>` placeholder; `/` redireciona para `/dashboard`
10. `[x]` Configurar `src/lib/money.ts` (Decimal.js: `formatBRL`, `parseBRL`, `sumMoney`, `moneySign`) e `src/lib/date.ts` (TZ `America/Sao_Paulo` via date-fns-tz)

**Critério de saída:**
- `pnpm dev` sobe sem erro
- `pnpm lint` e `pnpm typecheck` passam limpos
- AppShell renderiza com sidebar navegável entre rotas (mesmo que vazias)
- MoneyDisplay mostra `R$ 1.234,56` em verde e `−R$ 1.234,56` em vermelho terroso
- Supabase local responde em `:54321`

---

### ETAPA 2 — Autenticação e onboarding
*Objetivo: usuário consegue se cadastrar, fazer login e configurar a clínica pela primeira vez.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabelas `organizations` e `profiles` com RLS — migration `20260517135054_etapa2_auth_org_chart.sql`. **Desvio de ordenação:** `accounts` e `chart_of_accounts` (ETAPA 3 no roadmap) foram criadas aqui pois o onboarding precisa persistir a 1ª conta e semear o plano. ETAPA 3 só adiciona telas/CRUD sobre elas. RLS por org via função `auth_org_id()` SECURITY DEFINER (evita recursão); `profiles` isola por `id = auth.uid()`
2. `[x]` Configurar Supabase Auth (email/senha, sem social login no MVP) — `config.toml`: `enable_confirmations=false` no local, `site_url=localhost:3000`. Botão Google no mockup renderizado **desabilitado ("em breve")** p/ respeitar escopo no-social mantendo fidelidade visual
3. `[x]` Tela 16 — Login (`src/app/(auth)/login/page.tsx`) — fiel ao mockup (hero 2 painéis), `requestPasswordReset` via Mailpit
4. `[x]` Server Action de login + logout + redirect — `src/actions/auth.ts` (login, signup, logout, requestPasswordReset), validação zod, erros pt-BR
5. `[x]` Middleware Next.js — `src/middleware.ts` + `lib/supabase/middleware.ts`: refresh sessão, protege rotas, força `/onboarding` se sem org, redireciona logado-com-org p/ `/dashboard`
6. `[x]` Tela 17 — Onboarding wizard 4 passos (`src/app/onboarding/`) — fiel ao mockup; passo 3 = preview do plano (tipado em `lib/chart-of-accounts/corporis-plan.ts`)
7. `[x]` Seed do plano de contas Corporis ao concluir onboarding — função SQL `seed_corporis_chart` chamada por `complete_onboarding` (SECURITY DEFINER, atômico: org + profile + conta + plano). **Decisão signup:** rota `/signup` escondida (mockup não tem cadastro; critério exige fluxo do zero)

**Critério de saída:**
- Novo usuário completa o fluxo do zero: cria conta → onboarding → cai no dashboard vazio
- Logout redireciona para login
- Rota `/dashboard` sem login redireciona para `/login`
- `profiles` e `organizations` têm RLS que impede cross-tenant (verificar com `supabase db lint`)

**Status:** ✅ concluída e validada (smoke test backend 2026-05-17: signup → onboarding → seed 79 nós + hierarquia + conta criada; idempotência bloqueada; RLS isola — usuário B não vê dados do A). `pnpm lint`/`typecheck`/`build` limpos. UI das telas 16/17 fiel aos mockups. Falta validação visual no browser (abrir app vs mockup lado a lado).

---

### ETAPA 3 — Plano de contas + Contas e cartões
*Objetivo: estrutura mestre do sistema configurada. Tudo que vem depois depende disso.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabela `chart_of_accounts` com RLS — criada na migration `20260517135054_etapa2_auth_org_chart.sql` por dependência do onboarding
2. `[x]` `src/lib/dfc/` — helpers de leitura da hierarquia (nós pai, filhos, grupos, natureza) em `src/lib/dfc/chart-of-accounts.ts`
3. `[x]` Tela 13 — Plano de Contas — implementada em `/plano-de-contas` com árvore real, busca, expandir/recolher, edição inline, adicionar conta filha e ativar/desativar. Aviso bloqueando desativação por lançamentos futuros depende da tabela `transactions` (ETAPA 4)
   - Árvore hierárquica expansível com os 8 grupos
   - Editar nome e classificação de cada conta
   - Adicionar conta filha
   - Desativar (com aviso se tiver lançamentos futuros)
   - **Não** permite deletar — apenas desativar
4. `[x]` Migration: tabela `accounts` com RLS — criada na migration `20260517135054_etapa2_auth_org_chart.sql` por dependência do onboarding
5. `[x]` Tela 06 — Contas e Cartões — implementada em `/contas` com cards reais, resumo financeiro e diferenciação visual de conta bancária vs cartão. Até a Etapa 4/7, saldo atual = saldo inicial e faturas aparecem zeradas
   - Cards por conta com saldo (calculado do saldo inicial + lançamentos)
   - Diferenciar visualmente conta corrente de cartão de crédito
6. `[x]` Tela 07 — Modal de cadastro/edição de conta — implementado dentro de `/contas` com create/update via Server Action
   - Para cartão: campos de limite, dia de fechamento, dia de vencimento, conta de pagamento padrão

**Critério de saída:**
- Plano de contas carrega os 8 grupos e todas as subcontas da Corporis (verificar via seed)
- Consegue adicionar uma conta filha e ela aparece na árvore na posição correta
- Consegue cadastrar uma conta corrente e um cartão de crédito com campos diferentes
- Saldo inicial da conta aparece no card

---

### ETAPA 4 — Lançamentos manuais
*Objetivo: o usuário consegue registrar entradas, saídas e transferências. O núcleo do produto.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabelas `transactions` e `attachments` com RLS — migrations `20260517161658_etapa4_transactions_attachments.sql` e `20260517161912_etapa4_transfer_rpc.sql`; transferência ganha `transfer_group_id` + RPC atômica `create_manual_transfer`
2. `[x]` `src/lib/money.ts` — funções `sumMoney`, `formatBRL`, `parseBRL` com testes unitários em `tests/unit/money.test.ts`
3. `[x]` Server Actions: `createTransaction`, `updateTransaction`, `deleteTransaction` em `src/actions/transactions.ts`
   - Validação zod compartilhada (schema em `src/types/transaction.ts`)
   - Para transferência: criar 2 transações atômicas (saída + entrada) em uma única SQL transaction
4. `[x]` Tela 03 — Modal/painel lateral de novo lançamento — implementado como drawer em `/lancamentos`, com entrada/saída/transferência, categoria, contas, status, datas, preview de saldo inicial e anexos persistidos no Supabase Storage privado (`transaction-attachments`, migration `20260524180217_etapa4_transaction_attachments_storage.sql`)
   - Toggle Entrada / Saída / Transferência (muda campos disponíveis)
   - Combobox de categoria com busca e hierarquia do plano de contas
   - Preview do saldo da conta após o lançamento
   - Drag-and-drop de anexo
5. `[x]` Tela 02 — Listagem de lançamentos — implementada em `/lancamentos` com tabela, filtros client-side, paginação de 20 por página, edição/exclusão e total filtrado
   - Tabela com TanStack Table
   - Filtros: período, tipo, conta, categoria, status
   - Paginação (20 por página)
   - Abrir modal de edição ao clicar na linha
   - Total filtrado no rodapé
6. `[x]` Calcular e exibir saldo atual de cada conta (saldo inicial + soma de lançamentos realizados) — `/contas` já considera entradas, saídas e direção de transferências realizadas

**Critério de saída:**
- Criar entrada, saída e transferência — todos aparecem na listagem
- Filtrar por período retorna apenas lançamentos do período
- Transferência aparece nas duas contas mas não duplica no total de receitas/despesas
- Anexo salvo no Supabase Storage e visível na linha do lançamento via URL assinada
- Saldo da conta atualiza após cada lançamento
- `sumMoney` passa em todos os testes unitários (incluindo casos de arredondamento)

---

### ETAPA 5 — Dashboard
*Objetivo: visão executiva funcional com dados reais.*

**O que construir, nesta ordem:**

1. `[x]` Queries de agregação no servidor para o período atual (mês corrente como padrão):
   - Total de entradas do mês
   - Total de saídas do mês
   - Resultado do mês (entradas − saídas)
   - Saldo total de todas as contas
   - Variação % vs. mês anterior para cada KPI
2. `[x]` Componente `KPICard` (valor grande, label, delta vs. período anterior com seta)
3. `[x]` Tela 01 — Dashboard:
   - 4 KPI cards no topo
   - Gráfico de barras: Entradas vs. Saídas últimos 6 meses (Recharts com cores Corporis)
   - Gráfico de linhas: evolução do saldo (Recharts)
   - Donut: despesas por grupo do plano de contas
   - Lista "Últimos 5 lançamentos"
   - Cards de "Contas e cartões" com saldo
   - Lista "Próximos vencimentos de faturas"
4. `[x]` Seletor de período no dashboard (padrão: mês atual, troca via query param e re-fetcha os dados)

**Critério de saída:**
- Dashboard carrega em menos de 2s com 80 lançamentos de seed
- KPIs batem com a soma manual dos lançamentos (verificar com `seed-dev.ts`)
- Gráficos renderizam sem erros e com cores corretas da paleta Corporis
- Trocar o mês no seletor atualiza todos os dados

---

### ETAPA 6 — DFC
*Objetivo: o produto entrega seu valor central — o Demonstrativo de Fluxo de Caixa.*

**O que construir, nesta ordem:**

1. `[x]` `src/lib/dfc/calculate.ts` — função pura `calculateDFC(transactions, period, chartOfAccounts)`:
   - Agrupa por grupo e conta do plano de contas
   - Calcula linhas derivadas (Lucro Bruto, Líquido, Fluxos, Saldo Final)
   - Suporte a múltiplos meses (retorna mapa `{ month: DFCRow[] }`)
   - **Testes unitários obrigatórios** — testar cada linha calculada, incluindo edge cases (mês sem receita, mês sem despesa)
2. `[x]` `src/lib/dfc/vertical-analysis.ts` — transforma resultado do `calculateDFC` em percentuais
3. `[x]` Tela 08 — DFC:
   - Seletor de período (mês, trimestre, semestre, ano, customizado)
   - Tabela com colunas = meses selecionados + coluna Total
   - Grupos expansíveis/colapsáveis
   - Linhas calculadas em negrito com fundo diferenciado
   - Toggle AV% (troca valores absolutos por percentuais)
   - Botão exportar (gerar CSV simples primeiro; PDF fica para depois)

**Critério de saída:**
- `calculateDFC` passa em todos os testes unitários
- DFC anual mostra 12 colunas de meses com totais batem com soma manual
- Colapsar "DESPESAS OPERACIONAIS" oculta subgrupos 4.1, 4.2, 4.3 e suas contas
- AV% de Receita Bruta é sempre 100% (é a base do cálculo)
- Saldo Final do mês N = Saldo Final do mês N−1 + Fluxo de Caixa Livre do mês N

---

### ETAPA 7 — Importação com IA
*Objetivo: usuário consegue importar extrato e fatura de cartão com categorização automática.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabelas `imports` e `credit_card_invoices` com RLS — migration `20260518013816_etapa7_imports_invoices.sql`; também adiciona FKs pendentes em `transactions.credit_card_invoice_id` e `transactions.import_id`
2. `[x]` `src/lib/import/ofx-parser.ts` — parseia OFX e retorna `RawTransaction[]`
3. `[x]` `src/lib/import/csv-parser.ts` — parseia CSV com detecção de colunas
4. `[x]` `src/lib/import/pdf-parser.ts` — contrato zod + chamada Claude Vision/Anthropic para extrair fatura em JSON; requer `ANTHROPIC_API_KEY`
5. `[x]` `src/lib/import/dedup.ts` — detecta duplicatas por hash de `(conta, data, valor, descrição)`
6. `[x]` `src/lib/ai/categorize.ts` — memória de categorização por descrição confirmada, Claude Sonnet para novos padrões e fallback local por regras/keywords se a IA não estiver disponível
7. `[x]` API Route `POST /api/import/parse-[ofx|csv|pdf]` — CSV/OFX/PDF recebem arquivo, parseiam, deduplicam, sugerem categoria e registram importação
8. `[x]` Tela 04 — Upload: 3 zonas de upload, seleção de conta, resumo de parse e lista de importações recentes
9. `[x]` Tela 05 — Conciliação:
   - Tabela linha a linha com categoria sugerida pela IA (badge de confiança)
   - Editar categoria inline
   - Marcar como "ignorar"
   - Destacar duplicatas detectadas
   - Para fatura PDF: cabeçalho com dados da fatura + cada compra como linha
   - `[x]` Botão "Confirmar X lançamentos" → Server Action `confirmImport` persiste CSV/OFX/PDF, cria fatura do cartão e atualiza histórico; conciliação implementada no fluxo pós-upload
10. `[x]` Lógica de cartão ao confirmar fatura: `event_date` = data da compra, `cash_date` = vencimento da fatura, `credit_card_invoice_id` preenchido

**Critério de saída:**
- Upload de OFX real de um banco brasileiro parseia sem erro
- Upload de PDF de fatura real do Itaú ou Nubank extrai lançamentos corretamente
- IA sugere categoria para pelo menos 80% dos lançamentos com confiança > 0.7 após seed inicial
- Duplicatas são sinalizadas e não são importadas se o usuário não confirmar explicitamente
- Após confirmar fatura de cartão, os lançamentos aparecem no DFC no mês do vencimento

---

### ETAPA 8 — Orçamento
*Objetivo: usuário consegue planejar o ano e acompanhar execução.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabelas de orçamento com RLS — implementado como `budget_versions` e `budget_values` em `20260521010916_etapa8_budgets.sql`
2. `[x]` Server Actions: `saveBudget` cria/reusa versão ativa e persiste linhas; `createBudgetVersion` cria nova versão ativa, arquiva a anterior e preserva valores via RPC `create_budget_version`
3. `[x]` Tela 10 — Editor de orçamento:
   - Grid editável (linhas = contas do plano, colunas = 12 meses)
   - Inputs inline de moeda no grid
   - Ações: "Replicar para todos os meses", "Distribuir total igualmente", "Aplicar % de ajuste", "Usar média histórica"
   - Dropdown de versões
   - botão "Nova versão" com modal de nome, herança dos valores atuais e navegação para a versão criada
4. `[x]` `src/lib/dfc/budget-vs-actual.ts` — combina realizado de caixa com linhas de orçamento por mês e calcula favorabilidade
5. `[x]` Tela 09 — Orçado x Realizado:
   - Cards de resumo no topo (3 KPIs: receita, despesa, resultado)
   - Tabela com colunas: Orçado / Realizado / Variação R$ / Variação %
   - Variação com cor por favorabilidade (gastar menos é verde, faturar mais é verde)
   - implementada em `/orcamento`; editor movido para `/orcamento/editor`; validada visualmente pelo usuário

**Critério de saída:**
- Consegue salvar orçamento anual e criar uma segunda versão sem perder a primeira
- Variação de receita: positiva (realizou mais que orçado) aparece em verde
- Variação de despesa: negativa (realizou menos que orçado) aparece em verde
- "Replicar para todos os meses" preenche os 11 meses restantes com o mesmo valor

---

### ETAPA 9 — Projeção de caixa
*Objetivo: usuário vê para onde o caixa está indo nos próximos 90 dias.*

**O que construir, nesta ordem:**

1. `[x]` `src/lib/dfc/projection.ts`:
   - Coleta lançamentos `pending` com `cash_date` futuro
   - Calcula médias móveis dos últimos 3 meses por categoria
   - Combina com pesos por proximidade (mais próximo = mais confiável)
   - Aplica multiplicadores de cenário (conservador/realista/otimista)
2. `[x]` Tela 11 — Projeção de caixa:
   - Gráfico de linha: realizado (sólido) + projetado (tracejado) nos próximos 90 dias
   - Linha de saldo mínimo configurável (alerta visual se projeção cruza)
   - 3 cards: saldo em 30, 60, 90 dias
   - Tabela dia a dia com entradas/saídas/saldo
   - Toggle de cenários
   - implementada em `/projecao` com dados reais, pendentes + média histórica gradual e CSV; validada visualmente pelo usuário

**Critério de saída:**
- Projeção do dia seguinte ao dia atual usa apenas lançamentos `pending` reais
- Projeção de 60-90 dias inclui médias históricas
- Cenário conservador sempre projeta saldo menor que o realista
- Alerta visual dispara se saldo projetado fica abaixo do mínimo configurado

---

### ETAPA 10 — Consultor IA e acabamento
*Objetivo: produto polido e pronto para uso real.*

**O que construir, nesta ordem:**

1. `[x]` Migration: tabelas `ai_conversations` e `ai_messages` — RLS habilitada e policies por organização via `auth_org_id()`
2. `[x]` API Route `POST /api/ai/chat` — streaming SSE, persistência de conversa/mensagens, contexto financeiro real, fallback local quando `ANTHROPIC_API_KEY` não está configurada e consultas estruturadas determinísticas (`monthly_summary`, `expense_analysis`, `pending_cash_flow`) com metadata para cards/tabelas/actions
3. `[x]` Tela 12 — Consultor IA:
   - Layout split: histórico de conversas + área de chat
   - Streaming de respostas
   - Sugestões de perguntas iniciais
   - Capacidade de exibir gráficos e tabelas inline nas respostas da IA
   - implementada em `/consultor` com histórico persistido, input funcional, cards/tabelas/actions inline salvos em metadata; validada visualmente pelo usuário
4. `[x]` Tela 14 — Histórico de importações — implementada dentro de `/importacoes` com filtros por status/tipo, busca, KPIs, progresso, conta vinculada, revisão de importações pendentes e estado vazio; validada visualmente pelo usuário
5. `[x]` Tela 15 — Configurações — implementada em `/configuracoes` com apenas as seções funcionais visíveis: "Perfil e clínica" e "Segurança e acesso". As demais seções desenhadas (plano/cobrança, aparência, notificações, preferências financeiras, integrações, exportação e zona de perigo) ficam ocultas por `BACKLOG_SETTINGS_SECTIONS_ENABLED=false` até terem backend/comportamento real
6. `[x]` `ai_categorization_rules` — implementado como `category_memory` (migration `20260518030048_add_category_memory.sql`) com RLS por organização, aplicação antes da IA nas rotas de parse e persistência ao confirmar importações; `usage_count` agora é incrementado em vez de sobrescrito, com teste unitário
7. `[x]` Testes E2E com Playwright cobrindo fluxos críticos — Playwright configurado (`playwright.config.ts`, script `pnpm test:e2e`) com testes autolimpantes em `tests/e2e/core-flow.spec.ts`; artefatos ignorados em `.gitignore`/`biome.json`
   - `[x]` Login → novo lançamento → DFC carrega após criação
   - `[x]` Upload de CSV → conciliar → confirmar → lançamento aparece na listagem
   - `[x]` Orçamento → editor salva valores → Orçado x Realizado reflete o orçamento
8. `[x]` Revisão final de acessibilidade: foco visível global para elementos interativos, landmarks nomeados, labels/nomes acessíveis reforçados em buscas, chat e fluxo de lançamentos
9. `[x]` `scripts/seed-dev.ts` completo com 6 meses de dados realistas para demonstração — comando `pnpm db:seed`, idempotente por `external_id` prefixado com `seed-dev-v1`

**Critério de saída:**
- Todos os testes E2E passam
- `pnpm build` sem warnings
- Consultor IA responde em pt-BR com dados reais da organização
- Produto utilizável do zero ao DFC em menos de 10 minutos de onboarding

---

### Como iniciar cada etapa com o Claude Code

Ao começar uma nova sessão de trabalho, envie esta mensagem:

```
Estou na ETAPA [N] — [nome].
Leia o CLAUDE.md completo antes de começar.
Itens já concluídos nesta etapa: [liste os [x] marcados].
Quero trabalhar agora em: [item específico].
Confirme sua compreensão do contexto antes de escrever código.
```

Ao encerrar uma sessão:

```
Antes de terminar:
1. Rode pnpm lint e pnpm typecheck e corrija qualquer erro
2. Atualize os checkboxes desta seção no CLAUDE.md com o que foi concluído
3. Se criou tabela nova, confirme que RLS está habilitada
4. Se há testes faltando para lógica de DFC ou dinheiro, crie-os agora
```

---

## 17. Glossário rápido (para o Claude entender o domínio)

- **DFC**: Demonstrativo de Fluxo de Caixa. Mostra entradas e saídas de dinheiro por categoria.
- **Regime de caixa**: receita/despesa é reconhecida na data do efetivo movimento de dinheiro.
- **Regime de competência**: receita/despesa é reconhecida na data do fato gerador (não usamos no MVP).
- **AV (Análise Vertical)**: cada linha como % do total de entradas.
- **Orçado x Realizado**: comparação do planejado com o que efetivamente aconteceu.
- **Plano de contas**: estrutura hierárquica que classifica todas as entradas e saídas.
- **Conciliação**: ato de pareiar lançamentos do sistema com o extrato bancário.
- **Cash date vs event date**: data do efeito caixa vs data do fato. Crucial para cartão.
- **Cartão como caixa**: tratamos a despesa do cartão como saída na data do pagamento da fatura, não da compra.

---

*Última atualização: criação inicial. Versionar este arquivo é responsabilidade compartilhada.*

---
> Source: [corporisfisioterapiapilates-ops/corporis-finance](https://github.com/corporisfisioterapiapilates-ops/corporis-finance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
