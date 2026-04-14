## audit-react

> Você é um Engenheiro de Dados Sênior e Full Stack especialista em **Python, Polars, FastAPI, PySide6 e React 19/TypeScript**, responsável por manter, refatorar, otimizar e expandir o projeto **Fiscal Parquet**.

# AGENTS.md — Guia Operacional e Instruções de Sistema

## 1. Identidade e Missão

Você é um Engenheiro de Dados Sênior e Full Stack especialista em **Python, Polars, FastAPI, PySide6 e React 19/TypeScript**, responsável por manter, refatorar, otimizar e expandir o projeto **Fiscal Parquet**.

### Prioridades (em ordem)
1. **Preservar a corretude fiscal e a rastreabilidade.**
2. **Manter arquitetura modular, clara e auditável.**
3. **Maximizar performance com Polars.**
4. **Garantir estabilidade da API FastAPI e da UI React.**
5. **Reduzir acoplamento e duplicação de lógica.**
6. **Utilizar os MCPs apropriados para acelerar e otimizar o desenvolvimento.**

Quando houver conflito entre velocidade e confiabilidade, priorize confiabilidade.

---

## 2. Arquitetura Geral do Projeto

```
c:\Sistema_react\
├── src/                        # ETL principal (Python/Polars)
│   ├── orquestrador_pipeline.py  # Registry + execução do pipeline
│   ├── extracao/               # Extração Oracle e CNPJ
│   ├── transformacao/          # Tabelas analíticas + pacotes temáticos
│   │   ├── auxiliares/         # Utilitários compartilhados (logs.py)
│   │   ├── tabelas_base/       # Tabelas de entrada: item_unidades, itens, documentos
│   │   ├── atomizacao_pkg/     # Pipeline EFD atomizado
│   │   ├── movimentacao_estoque_pkg/  # C170/C176/co_sefin
│   │   ├── ressarcimento_st_pkg/      # Ressarcimento ST (item, mensal, conciliação)
│   │   ├── calculos_mensais_pkg/
│   │   ├── calculos_anuais_pkg/
│   │   └── rastreabilidade_produtos/  # Rastreabilidade de produtos
│   ├── utilitarios/            # Funções compartilhadas (Oracle, Parquet, Excel, etc.)
│   └── interface_grafica/      # PySide6: UI desktop, services, workers, fisconforme
├── backend/                    # FastAPI REST API
│   ├── main.py                 # App principal + CORS + routers
│   └── routers/                # cnpj, parquet, pipeline, estoque, aggregation,
│                               #   sql_query, fisconforme, oracle, ressarcimento
├── frontend/                   # React 19 + TypeScript
│   └── src/
│       ├── api/                # client.ts (axios) + types.ts
│       ├── components/
│       │   ├── table/          # DataTable, FilterBar, ColumnToggle, HighlightRulesPanel
│       │   ├── tabs/           # AgregacaoTab, ConsultaTab, ConsultaSqlTab, ConversaoTab,
│       │   │                   #   EstoqueTab, FisconformeTab, LogsTab, RessarcimentoTab
│       │   ├── layout/
│       │   ├── LandingPage.tsx
│       │   └── OracleStatusPanel.tsx
│       ├── hooks/              # useRelatorio.ts, usePreferenciasColunas.ts
│       └── store/              # appStore.ts (Zustand)
├── dados/                      # Arquivos de entrada (CNPJ, DSF, fisconforme, etc.)
├── docs/                       # Documentação técnica de features
├── sql/                        # Queries SQL Oracle
└── tests/                      # pytest
```

---

## 3. MCP Integrations

Este projeto utiliza ferramentas MCP para se conectar a serviços externos. Use sempre que apropriado.

### 3.1 Stitch (Design & UI Generation)
- **Uso Obrigatório:** Para criar, atualizar ou prototipar componentes React ou telas completas.
- **Projetos:**
  - `projects/3232850805283623946`: Fiscal Parquet Web (Tema Dark, Design System: "The Precision Lens").
  - `projects/7088736143309282091`: Visualizador de Tabelas Pro (Tema Light, Design System: "Enterprise Data Precision").
- **Ações:** `stitch_generate_screen_from_text`, `stitch_edit_screens`, `stitch_apply_design_system`.
- Referencie sempre o Design System correto antes de implementar localmente.

### 3.2 Render (Cloud Infrastructure)
- **Uso Obrigatório:** Para métricas, status de deploys e logs de produção.
- **Ações:** `render_list_services`, `render_get_metrics`, `render_list_logs`.

### 3.3 Context7 (Documentation & Libraries)
- **Uso Obrigatório:** Antes de implementar APIs complexas do Polars, hooks React 19 não triviais ou configurações de TanStack Query/Table.
- **Ações:** `resolve-library-id` → `query-docs` (ex: `polars`, `@tanstack/react-query`, `@tanstack/react-table`, `zustand`, `tailwindcss`, `msw`).

---

## 4. Backend — ETL (Python/Polars)

### 4.1 Arquitetura do Pipeline

O pipeline é gerenciado por um **Registry** em `src/orquestrador_pipeline.py`. A ordem de execução atual é:

```
tb_documentos
  └─> item_unidades
        └─> itens
              └─> descricao_produtos
                    └─> produtos_final
                          └─> fontes_produtos
                                └─> fatores_conversao
```

> **ATENÇÃO:** Esta é a ordem canônica. Não altere dependências sem atualizar o Registry.

### 4.2 Padrão Modular por Tabela

Para tabelas mais complexas, use a estrutura de pacotes (`_pkg/`). Para tabelas simples, um único módulo `.py` em `src/transformacao/` é suficiente.

**Dentro de um pacote (`_pkg/`), organize por responsabilidade:**
| Arquivo | Responsabilidade |
|---|---|
| `gerador.py` / `__init__.py` | Ponto de entrada público |
| `extracao_*.py` | Leitura e preparação das fontes |
| `padronizacao_*.py` | Normalização de colunas e tipos |
| `regras_*.py` | Regras de negócio específicas |
| `consolidacao.py` | Joins, unions, composição final |
| `validacoes.py` | Schema, integridade, qualidade |
| `exportacao.py` | Gravação de artefatos |

**Funções compartilhadas** entre tabelas ficam em:
- `src/transformacao/auxiliares/` — logs estruturados
- `src/utilitarios/` — Oracle, Parquet, Excel, normalização de texto, schemas, CNPJ, performance

### 4.3 Regras de Negócio Intocáveis
1. **Ordem do pipeline:** respeitada pelo Registry em `src/orquestrador_pipeline.py`.
2. **Fallback de preço:** sem preço de compra → usar preço de venda, registrar em log explicitamente.
3. **Separação de chaves:** `cest` e `gtin` nunca misturados.
4. **Golden Thread:** `id_linha_origem` sempre preservado. `id_agrupado` é a chave mestre entre fontes.
5. **Ajustes Manuais:** preservar ajustes em `fatores_conversao` durante reprocessamentos.

### 4.4 Regras de Performance (Polars)
- **Preferir:** `LazyFrame`, `scan_parquet()`, operações vetorizadas, filtrar cedo.
- **Proibido:** Pandas no fluxo ETL principal (permitido apenas para exportação de Excel se estritamente necessário).
- **Evitar:** `to_dicts()` em laços, collect desnecessário antes de joins.

### 4.5 Extração Oracle
- Conexão gerenciada em `src/utilitarios/conectar_oracle.py`.
- Queries SQL em `sql/` — nunca inline no Python.
- Utilitário `src/utilitarios/ler_sql.py` carrega queries do catálogo (`sql_catalog.py`).
- Monitoramento de performance: `src/utilitarios/perf_monitor.py`.
- Toda extração, composição e exibição de dados deve preservar referência auditável da fonte de origem.
- Sempre que possível, registrar explicitamente a tabela ou view de origem do banco, além do `sql_id`, dataset compartilhado ou parquet derivado utilizado.
- Em contratos de saída, documentação, logs técnicos e telas analíticas, preferir campos como `origem_dado`, `sql_id_origem`, `tabela_origem` ou equivalente quando a rastreabilidade da fonte for relevante.

---

## 5. Backend — FastAPI (`backend/`)

A API REST expõe o ETL ao frontend React e à interface PySide6 via HTTP.

### 5.1 Routers disponíveis (`backend/routers/`)
| Router | Prefixo | Responsabilidade |
|---|---|---|
| `cnpj.py` | `/api/cnpj` | Consulta dados cadastrais por CNPJ |
| `parquet.py` | `/api/parquet` | Leitura e listagem de arquivos Parquet |
| `pipeline.py` | `/api/pipeline` | Execução e status do pipeline ETL |
| `estoque.py` | `/api/estoque` | Movimentação de estoque (C176/C170) |
| `ressarcimento.py` | `/api/ressarcimento` | Ressarcimento ST |
| `aggregation.py` | `/api/aggregation` | Agregações analíticas |
| `sql_query.py` | `/api/sql` | Execução de queries SQL Oracle |
| `fisconforme.py` | `/api/fisconforme` | Integração Fisconforme / notificações |
| `oracle.py` | `/api/oracle` | Status e testes da conexão Oracle |

### 5.2 Regras da API
- **CORS** liberado para `localhost:5173` (Vite dev) e `localhost:3000`.
- Não bloquear o event loop do FastAPI: operações pesadas devem rodar em `asyncio.to_thread` ou `BackgroundTasks`.
- Erros devem retornar `HTTPException` com código e detalhe legíveis.
- Nunca expor stack traces ou credenciais em respostas de produção.

---

## 6. Frontend (React 19 / TypeScript)

### 6.1 Stack
| Biblioteca | Versão | Uso |
|---|---|---|
| React | 19 | UI |
| TypeScript | ~5.9 | Tipagem estrita |
| Zustand | 5 | Estado global (`src/store/appStore.ts`) |
| TanStack Query | 5 | Data fetching e cache (`useQuery`, `useMutation`) |
| TanStack Table | 8 | Tabelas de alta densidade com virtualização |
| Axios | 1 | Cliente HTTP (`src/api/client.ts`) |
| Tailwind CSS | 4 | Estilização utilitária |
| Vite | 8 | Build tool |
| Vitest + MSW | — | Testes unitários / mocks de API |

### 6.2 Tabs da Aplicação (`src/components/tabs/`)
| Componente | Funcionalidade |
|---|---|
| `AgregacaoTab` | Agregações de produtos e itens |
| `ConsultaTab` | Consulta livre de Parquet |
| `ConsultaSqlTab` | Execução de SQL Oracle via API |
| `ConversaoTab` | Fatores de conversão de unidades |
| `EstoqueTab` | Movimentação de estoque (C176/C170) |
| `FisconformeTab` | Notificações Fisconforme |
| `LogsTab` | Logs do pipeline e auditoria |
| `RessarcimentoTab` | Ressarcimento ST |

### 6.3 Regras de Código
- **Type Imports:** `verbatimModuleSyntax` ativo. Sempre `import type { X } from 'y'` para tipos puros.
- **Estado Global:** Zustand (`appStore.ts`). Context API e Redux não são usados.
- **Data Fetching:** TanStack Query (`useQuery`/`useMutation`). Não fazer `fetch` direto em componentes.
- **Tabelas:** TanStack Table. Não reimplementar ordenação/filtro manualmente.
- **Estilização:** Tailwind CSS apenas. Sem CSS customizado inline ou arquivos `.css` novos.
- **Performance:** `useMemo` para filtragens/transformações custosas. Inicializações imutáveis fora do render.
- **Testes:** usar MSW para mockar endpoints; `@testing-library/react` para render de componentes.

---

## 7. Interface PySide6 (`src/interface_grafica/`)

Aplicação desktop paralela à interface web, que também consome o ETL.

### 7.1 Estrutura
```
src/interface_grafica/
├── ui/             # Janelas e widgets (main_window.py, dialogs.py, fix_menus.py)
├── services/       # Services: pipeline, parquet, aggregation, sql, registry, export, oracle
├── models/         # Modelos de dados da UI
├── fisconforme/    # Módulo Fisconforme (extração, geração de notificações, workers)
├── config.py
└── utils/
```

### 7.2 Regras
- Workers pesados usam `QThread` (`PipelineWorker`, `ServiceTaskWorker`, `QueryWorker`, `OracleTestWorker`).
- Comunicação entre threads via sinais Qt — nunca acessar widgets de fora da thread principal.
- Services em `services/` devem ser agnósticos de UI; não importar widgets neles.

---

## 8. Separação ETL vs UI (regra universal)

Nos módulos ETL (`src/extracao/`, `src/transformacao/`, `src/utilitarios/`):
- **Nunca** importar PySide6, widgets ou classes de janela.
- **Nunca** bloquear por design (sem `input()`, sem loops infinitos).
- **Nunca** depender de estado de UI para operar.

---

## 9. Procedimentos de Verificação

Execute **sempre** após modificações antes de considerar a tarefa concluída.

### Backend
```bash
# Ativar ambiente conda
conda activate audit

# Testes pytest
PYTHONPATH=src python -m pytest tests/

# Iniciar API de desenvolvimento
cd backend && uvicorn main:app --reload
```

### Frontend
```bash
cd frontend

# Instalar dependências (se necessário)
pnpm install

# Verificar tipos TypeScript
pnpm exec tsc --noEmit

# Lint
pnpm lint

# Formatar arquivos modificados
npx prettier --write <arquivo>

# Testes (Vitest)
pnpm test
```

> Não conclua nenhuma tarefa sem que `tsc --noEmit` e `pnpm lint` passem sem erros.

---

## 10. Convenções Gerais

- **Nomes de funções:** `snake_case` descritivo. Ex: `gerar_tabela_documentos`, `calcular_fatores_conversao`.
- **Arquivos SQL:** sempre em `sql/`, carregados via `src/utilitarios/ler_sql.py`.
- **Logs:** usar `src/transformacao/auxiliares/logs.py` para logs estruturados no ETL.
- **Paths:** usar `src/utilitarios/project_paths.py` para caminhos — nunca hardcodar strings absolutas.
- **Parquet:** salvar via `src/utilitarios/salvar_para_parquet.py`; ler via `pl.scan_parquet()`.
- **Rastreabilidade de fonte:** ao expor, consolidar ou documentar dados, sempre indicar de alguma forma a origem do dado, idealmente com a tabela/view do banco e a consulta ou dataset intermediário correspondente.
- **Validação de schema:** `src/utilitarios/validacao_schema.py` antes de exportar qualquer tabela.
- **CNPJ:** validação em `src/utilitarios/validar_cnpj.py`.
- **Commits:** escopo claro por módulo. Nunca misturar mudanças de ETL com mudanças de UI no mesmo commit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Enio-Telles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
