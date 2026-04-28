## farm-erp

> Coffee Farm ERP é um sistema integrado de gestão para fazendas de café, cobrindo operações comerciais, financeiras, de estoque, produção e folha de pagamento. O projeto foi estruturado para escalar de forma modular e sustentável.

# Coffee Farm ERP - Diretrizes do Projeto

## Visão Geral

Coffee Farm ERP é um sistema integrado de gestão para fazendas de café, cobrindo operações comerciais, financeiras, de estoque, produção e folha de pagamento. O projeto foi estruturado para escalar de forma modular e sustentável.

## Stack Completa

**Backend:**
- FastAPI (Python 3.11+)
- SQLAlchemy ORM
- Alembic para migrations
- PostgreSQL

**Frontend:**
- Next.js 14 com App Router
- TypeScript (sem `any`)
- Tailwind CSS
- shadcn/ui para componentes

**Deploy:**
- Railway

## Idioma

- **Código:** Inglês (variáveis, funções, classes, comentários)
- **Interface e Mensagens:** Português
- **Documentação:** Português

## Arquitetura Backend

### Fluxo de Requisição

```
Router → Service → Repository → Database
```

**Regras imutáveis:**
- Router valida entrada (opcional), chama Service, retorna resposta
- Service contém lógica de negócio, orquestra Repositories, valida regras
- Repository acessa banco, nunca contém lógica
- **Nunca pular camadas:** Router não fala direto com Repository
- Toda integração entre módulos ocorre no Service Layer

### Validação e Erros

- Validação de entrada com Pydantic
- HTTPException com mensagens em português
- Status codes HTTP apropriados
- Sem exposição de stack trace ao cliente

### Banco de Dados

**Campos obrigatórios em toda tabela:**
- `id` UUID, chave primária
- `created_at` timestamp, preenchido automaticamente
- `updated_at` timestamp, atualizado automaticamente

**Soft Delete obrigatório em tabelas de negócio:**
- Campo `deleted_at` nullable (NULL = ativo)
- Sempre filtrar por `deleted_at IS NULL` nas queries

**Nomenclatura:**
- Tabelas: plural, snake_case (ex: `customers`, `order_items`)
- Colunas: snake_case (ex: `created_at`, `total_amount`)
- Constraints: usar prefixos (ex: `fk_`, `idx_`, `ck_`)
- Foreign keys com índice automático

**Migrations:**
- Todas via Alembic, nunca DDL avulso
- Versionadas e reproduzíveis
- Comando: `alembic upgrade head`

**Seeds:**
- Scripts SQL em `scripts/seed.sql`
- Dados realistas e consistentes
- População via: `make reset-db` ou `python scripts/reset_db.py`

## Arquitetura Frontend

### Fluxo de Dados

```
Page → Service → API ← Backend
```

**Regras imutáveis:**
- Pages (rotas) chamam Services, nunca acessam API diretamente
- Services importam funções API e orquestram chamadas
- Componentes são "burros": recebem props, emitem callbacks
- **Sem fetch direto em componentes:** usar `/services`

### Formulários

- React Hook Form para gerenciamento de estado
- Zod para validação de schema
- Mensagens de erro em português
- Loading states em toda ação assíncrona
- Toasts para feedback (sucesso, erro, aviso)

### Componentes

- shadcn/ui para componentes base
- Componentes customizados em `components/modules`
- Componentes de layout em `components/layout`
- Responsabilidade única por componente

### TypeScript

- Sem `any`
- Tipagem estrita
- Interfaces em `types/`
- Tipos descritivos

## Nomenclatura

### Backend Python

- **Funções:** `camelCase` (ex: `get_customer_by_id`)
- **Classes:** `PascalCase` (ex: `CustomerService`)
- **Constantes:** `UPPER_SNAKE_CASE` (ex: `MAX_RETRY_ATTEMPTS`)
- **Módulos:** `snake_case` (ex: `customer_service.py`)

### Frontend TypeScript

- **Componentes:** `PascalCase` (ex: `CustomerForm.tsx`)
- **Funções:** `camelCase` (ex: `formatCurrency`)
- **Interfaces:** `PascalCase` prefixado com `I` ou sufixado com `Type` (ex: `ICustomer` ou `CustomerType`)
- **Arquivo:** kebab-case ou PascalCase para componentes (ex: `customer-form.tsx` ou `CustomerForm.tsx`)

## Operações Obrigatórias

### Toda Venda (Comercial)

1. Baixa no Estoque
2. Gera Fatura (Faturamento)
3. Lança Conta a Receber (Financeiro)
4. Gera Movimentação (Financeiro)

### Toda Compra (Compras)

1. Entrada no Estoque
2. Lança Conta a Pagar (Financeiro)
3. Gera Movimentação (Financeiro)

### Toda Folha de Pagamento

1. Calcula salários, descontos, extras
2. Lança Contas a Pagar (Financeiro)
3. Gera Movimentação por funcionário (Financeiro)

### Produção de Safra (PCP)

1. Baixa insumos do Estoque
2. Gera Movimentação (Financeiro)
3. Simula resultado com variação de qualidades
4. Insere café produzido no Estoque
5. Gera Movimentação (Financeiro)

## Controle de Qualidade

- Build deve passar sem erros: `npm run build` (frontend) e uvicorn --reload (backend)
- Sem TypeScript errors no frontend
- Sem Python linting errors no backend
- Migrations devem ser reversíveis

## Documentação

Cada agente documenta seu trabalho:
- Backend: `docs/backend/[modulo].md`
- Frontend: `docs/frontend/[modulo].md`
- Database: `docs/database/schema.md`

## Próximas Etapas

1. ✅ Estrutura de pastas e documentação
2. ✅ Inicializar repositórios (Poetry para backend, npm para frontend)
3. Configurar banco PostgreSQL
4. Implementar módulo Auth
5. Implementar módulos por ordem de dependência

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gutvono) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
