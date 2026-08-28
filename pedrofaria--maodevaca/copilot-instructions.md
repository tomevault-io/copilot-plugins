## maodevaca

> Guia de convenções para agentes trabalhando neste repositório.

# AGENTS.md — Mão de Vaca

Guia de convenções para agentes trabalhando neste repositório.

## O que é

**Mão de Vaca** é um app desktop de finanças pessoais (UI em **pt-BR**) construído com Wails v2.
Acompanha contas a pagar recorrentes (mensalidades, aluguel, assinaturas) e entradas de crédito,
com relatórios mensais/anuais.

## Stack

| Camada | Tecnologia |
|---|---|
| Shell desktop | Wails v2.10.1 (Go 1.25) |
| Banco de dados | SQLite puro-Go (`modernc.org/sqlite` v1.56) — **sem CGO** |
| Frontend | Vue 3.5 + Vite 6 + TypeScript |
| UI | Nuxt UI v4 + Tailwind CSS v4 (classes utilitárias direto no template) |
| Rotas | vue-router (hash mode) |

Dados ficam em `%APPDATA%\maodevaca\maodevaca.db` (não versionado, fora do repo).

## Comandos

```bash
wails dev        # desenvolvimento com hot reload (frontend + backend)
wails build      # build de produção (gera o .exe)
npm run build    # type-check (vue-tsc --noEmit) + build do frontend
go test ./...    # testes do backend (backend_test.go)
```

- Com `wails dev` rodando, o debug no navegador fica em `http://localhost:34115` (Vite em porta 5173+).
- Ao adicionar/renomear métodos Go do `App`, os bindings são regenerados em `frontend/wailsjs/` e o
  frontend usa `frontend/src/lib/api.ts` (wrapper tipado sobre `window.go.main.App`).

## Semântica de domínio (CRÍTICO — não confundir)

- **"Contas"** = contas a pagar / despesas **recorrentes** (com `dueDay`, `amount`, `groupId`).
  **NUNCA** significa conta corrente de banco. Grupos (`groups`) organizam essas contas
  (ex.: "Casa", "Cabo Frio", "Pessoal").
- **"Créditos"/"Entradas"** = entradas de dinheiro (`incomes`), cada uma ligada a uma **fonte**
  (`income_sources`, ex.: "Salário iFood", "Aluguel").
- **"Pagamentos"** (`payments`) = marcação de que uma conta foi paga em um mês específico
  (chave: account + year + month). Uma conta pode ser "paga" e depois "desfeita" (unpay).

## Arquitetura

**Backend (Go, package `main`):**
- `main.go` — bootstrap Wails (janela 1200×800, bind do `App`)
- `app.go` — struct `App` + handlers de grupos/contas expostos ao frontend
- `db.go` — inicialização do SQLite e schema
- `models.go` — structs (Group, Account, Payment, IncomeSource, Income, MonthSummary, YearSummary)
- `incomes.go` — handlers de fontes e entradas
- `reports.go` — agregações mensais/anuais (MonthSummary, YearSummary)

**Frontend:**
- `frontend/src/views/` — uma view por rota:
  - `ReportsView.vue` — visualização mensal/anual (tela inicial `/`)
  - `ContasView.vue` — CRUD de contas e grupos (tela de GERENCIAMENTO)
  - `TransacoesView.vue` — operacional: pagar contas + lançar entradas do mês
  - `FontesView.vue` — CRUD de fontes de crédito (gerenciamento)
- `frontend/src/lib/` — `api.ts` (bridge tipada), `types.ts`, `format.ts` (money/date pt-BR),
  `state.ts` (mês/ano em exibição), `icons.ts`, `colorMode.ts`
- `frontend/src/router.ts` — rotas hash (`#/contas`, `#/transacoes`, ...)

## Convenções de UI

- Todo texto visível em **pt-BR** (inclusive comentários de código, que seguem o padrão do repo).
- **Separação de telas**: CRUD/gerenciamento (Contas, Fontes) fica em rotas próprias, separado das
  telas operacionais de lançamento (Transações) — não misturar na mesma página. Dashboard resume tudo.
- **Código de cores**: pagamentos = vermelho (`red-*`), entradas = verde (`emerald-*`).
- Listagens de cards usam grid de 2 colunas (`grid grid-cols-1 sm:grid-cols-2 gap-2`) com cabeçalhos
  em linha inteira (`sm:col-span-2`). Barra de total usa `grid grid-cols-[1fr_auto_1fr]` (label | ação | total).
- Modais de confirmação (`UModal`) para ações destrutivas (excluir entrada, remover pagamento).
- Valores monetários: `float64` em **reais** (ex.: 8340 = R$ 8.340,00), formatados com `money()` de `format.ts`
  (Intl `pt-BR`, BRL). Datas em ISO `YYYY-MM-DD`.
- Componentes globais Nuxt UI (`UButton`, `UInput`, `UModal`, `UIcon`...) são auto-importados — não importar manualmente.

## Git

- Remote: `git@github.com:pedrofaria/maodevaca.git` (SSH), branch padrão `main`.
- `.gitignore` cobre: `maodevaca.exe`, `build/bin/`, `build/windows/installer/tmp/`, `node_modules/`, `frontend/dist/`.

## Armadilhas conhecidas

- **NÃO rode comandos git que tocam arquivos do working tree (`git add`, `git checkout`, `git stash`) com
  `wails dev` ativo** — a enxurrada de eventos do watcher (incluindo `vite.config.ts`) faz o Vite reiniciar
  em cascata e derruba o processo do dev. Pausar o dev antes de operações git.
- `frontend/auto-imports.d.ts` e `frontend/components.d.ts` são **regenerados** pelo Vite e podem aparecer
  como "modificados" após rodar o dev — costuma ser ruído de CRLF (conteúdo idêntico); conferir com `git diff`
  antes de commitar.
- O processo `wails dev` pode morrer sozinho (killed) — se a janela sumir, verificar `process` e reiniciar.
- Não commitar segredos; `.env` e credenciais ficam fora do repo.

---
> Source: [pedrofaria/maodevaca](https://github.com/pedrofaria/maodevaca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
