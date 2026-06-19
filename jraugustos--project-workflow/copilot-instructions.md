## project-workflow

> >


# Project Workflow — Framework de Desenvolvimento Estruturado

Framework em **5 etapas** para desenvolver projetos com IA sem perder qualidade.
O principio central: **cada etapa roda em uma conversa separada**, com limpeza de contexto entre elas.
O output de cada etapa e um documento que alimenta a proxima.

## Os 5 Problemas que Este Framework Resolve

1. **Over-engineering** — a IA complica o que poderia ser simples
2. **Reinventar a roda** — criar do zero em vez de usar padroes existentes
3. **Limites de conhecimento do modelo** — docs mais recentes que o training cutoff
4. **Codigo duplicado** — componentes repetidos por falta de visao global
5. **Nao saber implementar o que foi pedido** — bloqueio por falta de pesquisa

## Principio: Context Window Discipline

```
CONVERSA 1 — Pesquisa
  Pesquisar padroes, refs, docs → gerar PRD.md
  /clear (ou nova conversa)

CONVERSA 2 — Planejamento
  Ler PRD.md → gerar SPEC.md (arquitetura + decisoes)
  /clear (ou nova conversa)

CONVERSA 3 — Quebra da Spec
  Ler SPEC.md → gerar TASKS.md (tarefas atomicas ordenadas)
  /clear (ou nova conversa)

CONVERSA 4..N — Build (uma por task)
  Ler TASKS.md → implementar Task Txx → testar → commit
  /clear (ou nova conversa)
  Ler TASKS.md → implementar Task Txx+1 → testar → commit
  ...
```

A qualidade do input determina a qualidade do output.
Se a spec nao e clara, a IA vai implementar do jeito dela — e provavelmente vai sair ruim.

---

## Etapa 0: Setup do Projeto

Antes de comecar, estruture o workspace:

```
projeto/
├── docs/
│   ├── PRD.md              ← Output da Etapa 1
│   ├── SPEC.md             ← Output da Etapa 2
│   ├── TASKS.md            ← Output da Etapa 3
│   ├── RESEARCH-LOG.md     ← Notas brutas da pesquisa (opcional)
│   └── DECISIONS.md        ← ADRs simplificados (opcional)
├── CLAUDE.md               ← Instrucoes persistentes para o Claude Code
└── src/                    ← Codigo (Etapa 4+)
```

### CLAUDE.md inicial

Crie um `CLAUDE.md` na raiz com contexto minimo do projeto:

```markdown
# [Nome do Projeto]

## Contexto
[1-2 frases sobre o que e o projeto]

## Stack
[Tecnologias escolhidas ou a definir]

## Convencoes
- [Patterns de codigo, naming, etc.]

## Docs de referencia
- docs/PRD.md — Requisitos do produto
- docs/SPEC.md — Especificacao tecnica
- docs/TASKS.md — Tarefas de implementacao
```

---

## Etapa 1: Pesquisa (Research) — CONVERSA 1

**Objetivo:** Entender o problema, encontrar padroes existentes, estudar implementacoes de referencia.
**Output:** `docs/PRD.md`
**Duracao tipica:** 1 conversa

### O que fazer nesta etapa

1. **Definir o problema claramente**
   - O que precisa ser construido?
   - Quem vai usar?
   - Quais sao os requisitos funcionais e nao-funcionais?

2. **Pesquisar padroes e implementacoes existentes**
   - Buscar no Stack Overflow, GitHub, docs oficiais
   - Importar repos de referencia para o Claude analisar (clonar em pasta temporaria)
   - Preferir padroes documentados e provados — nunca reinventar a roda
   - Se a feature depende de docs mais recentes que o modelo conhece, usar WebSearch/WebFetch

3. **Analisar trade-offs**
   - Listar abordagens possiveis
   - Comparar pros/contras de cada uma
   - Decidir a abordagem antes de planejar

### Tecnicas de pesquisa

| Tecnica | Quando usar |
|---|---|
| **Clonar repo de referencia** | Projeto open-source faz algo similar — clone em `/tmp` e peca ao Claude para estudar o padrao |
| **WebSearch para docs recentes** | Lib/framework mais nova que o cutoff do modelo |
| **Ler source code de libs** | Precisa entender como algo funciona internamente |
| **Analisar issues do GitHub** | Quer saber problemas comuns de uma abordagem |
| **Consultar CHANGELOG** | Precisa de features especificas de uma versao |

### Template do PRD.md

Consulte `references/templates.md` secao "PRD" para o template completo.

O PRD deve ser **conciso** — um resumo do que foi pesquisado, nao um despejo de tudo que foi lido.
Se o PRD passar de 3-4 paginas, esta longo demais. Resuma mais.

### Ao terminar

1. Gere o `docs/PRD.md` com todo o conhecimento acumulado
2. Revise com o usuario: "O PRD esta completo? Falta algo?"
3. **Limpe o contexto** — `/clear` no Claude Code, nova conversa no Cowork
4. Na proxima conversa, comece referenciando o PRD

---

## Etapa 2: Planejamento (Spec) — CONVERSA 2

**Objetivo:** Transformar o PRD em uma especificacao tecnica com arquitetura e decisoes.
**Input:** `docs/PRD.md`
**Output:** `docs/SPEC.md`
**Duracao tipica:** 1 conversa

### O que fazer nesta etapa

1. **Ler o PRD.md** como ponto de partida
2. **Definir arquitetura** — componentes, fluxo de dados, integracoes
3. **Tomar decisoes tecnicas** — stack, libs, patterns, estrutura de pastas
4. **Definir contratos** — APIs, modelo de dados, interfaces entre componentes
5. **Identificar riscos** — o que pode dar errado e como mitigar

A SPEC e o "raciocinio tecnico" — ela explica COMO o projeto vai ser construido.
Ela nao precisa listar tarefas ainda (isso e a Etapa 3).

### Template do SPEC.md

Consulte `references/templates.md` secao "SPEC" para o template completo.

A spec deve ser **tatica e precisa**. Se voce nao deixar claro o que quer, a IA vai implementar
do jeito dela — e voce vai reclamar que saiu errado. Quanto mais especifica a spec, melhor o codigo.

### Ao terminar

1. Gere o `docs/SPEC.md` completo
2. Revise com o usuario: "A arquitetura faz sentido? As decisoes estao certas?"
3. **Limpe o contexto** — `/clear` no Claude Code, nova conversa no Cowork
4. Na proxima conversa, comece pela SPEC para quebrar em tasks

---

## Etapa 3: Quebra em Tasks — CONVERSA 3

**Objetivo:** Transformar a SPEC em tarefas atomicas, ordenadas e implementaveis individualmente.
**Input:** `docs/SPEC.md`
**Output:** `docs/TASKS.md`
**Duracao tipica:** 1 conversa

Esta etapa e crucial. Cada task vai virar uma conversa de implementacao separada,
entao precisa ser autocontida — com tudo que o Claude precisa para implementar sem depender do contexto anterior.

### O que fazer nesta etapa

1. **Ler a SPEC.md**
2. **Quebrar em tarefas atomicas** — cada task deve ser implementavel em uma unica conversa
3. **Ordenar por dependencia** — o que precisa existir antes de cada task
4. **Descrever cada task com precisao** — arquivos envolvidos, o que criar/alterar, criterios de aceitacao
5. **Agrupar em fases logicas** — fundacao, core, integracao, polish

### Regras para uma boa task

Cada task deve ter:
- **Descricao clara** do que implementar (nao o que pesquisar ou decidir — isso ja foi feito)
- **Arquivos envolvidos** — quais criar, quais alterar
- **Criterio de aceitacao** — como saber que esta pronta (teste, comportamento, output)
- **Dependencias** — quais tasks precisam estar prontas antes
- **Contexto necessario** — trechos da SPEC que a task precisa (para nao precisar ler a spec inteira)

Uma task boa e como um ticket de Jira bem escrito: alguem (ou uma IA) pega e implementa sem precisar perguntar nada.

### Template do TASKS.md

```markdown
# TASKS — [Nome do Projeto/Feature]

> Baseado em: docs/SPEC.md
> Total: XX tasks | Status: 0/XX concluidas

## Fase A: Fundacao

### T01 — [Nome descritivo da task]
- **O que fazer:** [Descricao precisa do que implementar]
- **Arquivos:** `src/...` (criar), `src/...` (alterar)
- **Contexto da SPEC:** [Trecho relevante ou referencia a secao]
- **Criterio de aceitacao:** [Como validar — teste, comportamento, etc.]
- **Dependencias:** nenhuma
- **Status:** [ ] pendente

### T02 — [Nome descritivo]
- **O que fazer:** [Descricao]
- **Arquivos:** `src/...`
- **Contexto da SPEC:** [Trecho relevante]
- **Criterio de aceitacao:** [Como validar]
- **Dependencias:** T01
- **Status:** [ ] pendente

## Fase B: Core

### T03 — [Nome descritivo]
...

## Fase C: Integracoes

### T04 — [Nome descritivo]
...

## Fase D: Polish & Testes

### T05 — [Nome descritivo]
...
```

### Ao terminar

1. Gere o `docs/TASKS.md` completo
2. Revise com o usuario: "As tasks fazem sentido? A ordem esta certa? Falta algo?"
3. **Limpe o contexto** — `/clear` no Claude Code, nova conversa no Cowork
4. A partir daqui, cada task e uma conversa separada

---

## Etapa 4: Build (Implementacao) — CONVERSA 4, 5, 6...

**Objetivo:** Implementar uma task por conversa, com contexto limpo e focado.
**Input:** `docs/TASKS.md` (task especifica) + `CLAUDE.md`
**Output:** Codigo implementado + task marcada como concluida

### Principio fundamental

**Uma task = uma conversa.**

Isso porque:
- O contexto inteiro fica livre para codigo (nao tem pesquisa/planejamento poluindo)
- A task ja tem tudo que o Claude precisa saber (arquivos, criterio, contexto da spec)
- Se algo der errado, voce perde so o contexto daquela task, nao de todo o projeto

### Fluxo de cada conversa de build

```
1. Abrir conversa limpa
2. Prompt: "Leia docs/TASKS.md, task Txx. Implemente seguindo as instrucoes."
3. Claude implementa
4. Testar (rodar testes, verificar comportamento)
5. Se passou → commit + marcar task como concluida no TASKS.md
6. Se falhou → corrigir na mesma conversa (o contexto ainda tem o codigo fresco)
7. /clear (ou nova conversa para a proxima task)
```

### Prompt padrao para iniciar uma task

```
Leia docs/TASKS.md, task T[XX].
Leia tambem CLAUDE.md para contexto do projeto.
Implemente exatamente o que a task descreve.
Ao terminar, rode os testes e me mostre o resultado.
```

### Quando a implementacao revela problemas

| Tipo de problema | Acao |
|---|---|
| **Pequeno** (bug, ajuste) | Resolve na mesma conversa, anota em DECISIONS.md |
| **Medio** (task mal definida) | Pausa, atualiza TASKS.md, continua em nova conversa |
| **Grande** (arquitetura errada) | Volta para Etapa 2 (SPEC), replaneja, regera tasks |
| **Pesquisa necessaria** | Volta para Etapa 1, pesquisa o que falta, atualiza PRD → SPEC → TASKS |

Nao resolva problemas arquiteturais "na marra" durante o build.
E melhor voltar uma etapa do que acumular divida tecnica.

### Atualizar TASKS.md apos cada task

Marque tasks concluidas diretamente no TASKS.md:

```markdown
### T01 — Setup do projeto
- **Status:** [x] concluida (2026-04-03)
```

Isso cria um log de progresso e ajuda a saber onde voce parou.

---

## Guia de Uso por Contexto

### No Claude Code (CLI)

```bash
# Etapa 1: Pesquisa (conversa 1)
claude "Vamos pesquisar padroes para [descreva o projeto]. Busque refs no GitHub, docs oficiais..."
# ... pesquisa acontece ...
claude "Consolide tudo em docs/PRD.md seguindo o template"
/clear

# Etapa 2: Spec (conversa 2)
claude "Leia docs/PRD.md e crie a spec tecnica em docs/SPEC.md"
/clear

# Etapa 3: Quebra em tasks (conversa 3)
claude "Leia docs/SPEC.md e quebre em tasks atomicas em docs/TASKS.md"
/clear

# Etapa 4: Build — task por task (conversas 4, 5, 6...)
claude "Leia docs/TASKS.md, task T01. Implemente."
# ... implementa, testa, commit ...
/clear
claude "Leia docs/TASKS.md, task T02. Implemente."
# ... e assim por diante
```

### No Cowork (Claude Desktop)

Cada etapa e uma **conversa separada**. Ao iniciar cada conversa:
- Diga em qual etapa/task esta
- Aponte para os docs relevantes (PRD, SPEC, TASKS)
- Seja explicito: "Estou na Etapa 4, task T03"

### Para features em projeto existente

O framework escala para features individuais:

```
projeto/
├── docs/
│   ├── PRD.md                    ← Projeto geral
│   ├── SPEC.md                   ← Spec geral
│   ├── TASKS.md                  ← Tasks gerais
│   └── features/
│       ├── auth/
│       │   ├── PRD.md            ← PRD da feature
│       │   ├── SPEC.md           ← Spec da feature
│       │   └── TASKS.md          ← Tasks da feature
│       └── payments/
│           ├── PRD.md
│           ├── SPEC.md
│           └── TASKS.md
```

### Para projetos pequenos (scripts, automacoes)

O framework ainda funciona, so que mais rapido. As etapas podem levar minutos em vez de horas.
Mesmo assim, os documentos existem — voce sempre pode voltar e consultar o raciocinio.

---

## Resumo Visual do Framework

```
┌─────────────────────────────────────────────────┐
│  ETAPA 1: PESQUISA          [Conversa 1]        │
│  Input: Problema/ideia                          │
│  Fazer: Pesquisar padroes, refs, docs           │
│  Output: docs/PRD.md                            │
│  Depois: /clear                                 │
└─────────────────────┬───────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  ETAPA 2: SPEC               [Conversa 2]       │
│  Input: docs/PRD.md                             │
│  Fazer: Arquitetura, decisoes, contratos        │
│  Output: docs/SPEC.md                           │
│  Depois: /clear                                 │
└─────────────────────┬───────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  ETAPA 3: QUEBRA EM TASKS    [Conversa 3]       │
│  Input: docs/SPEC.md                            │
│  Fazer: Tasks atomicas, ordenadas, autocontidas │
│  Output: docs/TASKS.md                          │
│  Depois: /clear                                 │
└─────────────────────┬───────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  ETAPA 4: BUILD              [Conversa 4..N]    │
│  Input: docs/TASKS.md (task Txx)                │
│  Fazer: Implementar 1 task por conversa         │
│  Output: Codigo + task marcada como concluida   │
│  Depois: /clear → proxima task                  │
└─────────────────────────────────────────────────┘
```

---

## Integracao com Outras Skills

| Quando | Skill |
|---|---|
| Precisa documentar o projeto completo apos o build | `project-knowledge-hub` |
| Quer conselho estrategico sobre o projeto | `carreira-advisor` |
| Precisa planejar a semana de desenvolvimento | `rotina-planner` |
| Quer registrar o progresso no diario | `hub-diario` |

---

## Checklist Rapido

Antes de comecar a codar (Etapa 4), confirme:

- [ ] Pesquisou padroes existentes? (nao esta reinventando a roda?)
- [ ] PRD.md existe e foi revisado?
- [ ] SPEC.md existe com arquitetura e decisoes?
- [ ] TASKS.md existe com tasks atomicas ordenadas?
- [ ] Cada task tem descricao, arquivos, criterio e dependencias?
- [ ] CLAUDE.md tem o contexto minimo do projeto?
- [ ] Limpou o contexto entre cada etapa?

Se algum item esta faltando, volte para a etapa correspondente.

---
> Source: [jraugustos/project-workflow](https://github.com/jraugustos/project-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
