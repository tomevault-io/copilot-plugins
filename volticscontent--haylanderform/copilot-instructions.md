## haylanderform

> O **Haylander Form** é um CRM/ERP + Automação de WhatsApp focado na contabilidade de MEIs.

# Setup e Contexto Global

O **Haylander Form** é um CRM/ERP + Automação de WhatsApp focado na contabilidade de MEIs.

Temos um Monorepo informal composto por:
1. **Frontend Admin:** Em Next.js App Router (pasta principal atual).
2. **Bot Backend:** Contido em `/bot-backend` (Node.js, Express, Baileys/Evolution API, BullMQ).
3. **Knowledge Base:** Baseado no método Karpathy em `knowledge-base/` (wiki + memory management).

---

## 🧠 NOVO: Knowledge Base Management (Método Karpathy)

**Local:** `D:\Códigos\Haylander\haylanderform\knowledge-base/`

Este é seu **persistent knowledge base** que cresce conforme o desenvolvimento avança. Você (Claude Code) mantém, evolui e atualiza automaticamente.

### Propósito

Documentar de forma viva:
- Arquitetura de sistema (frontend, backend, integrações)
- Decisões de design e trade-offs
- Fluxos de negócio (Contabilidade, MEI, Serpro, Integra Contador)
- Automações (WhatsApp, BullMQ, Agente Apolo)
- Segurança (Serpro, Certificados, OAuth)
- Integrações (Serpro, Caixa Postal, Calima)

### Estrutura

```
knowledge-base/
├── CLAUDE.md              ← (este arquivo expandido)
├── raw/                   ← Documentação bruta (imutável)
│   ├── docs/             ← Referências (plan.master.*.md, etc)
│   └── assets/
├── git/                   ← Config GitHub (automático)
│   └── config.json
└── wiki/                  ← Knowledge base vivo
    ├── index.md           ← Índice master
    ├── log.md             ← Append-only operational log
    ├── overview.md        ← Síntese viva da arquitetura
    ├── migrations/        ← Relatórios temporais (semanais)
    ├── architecture/      ← Decisões e designs
    ├── features/          ← Features documentadas
    ├── integrations/      ← Integrações (Serpro, Caixa, etc)
    ├── security/          ← Segurança, certificados, OAuth
    ├── workflows/         ← Fluxos de negócio
    └── decisions/         ← ADRs (Architecture Decision Records)
```

### Como Funciona

**Você (Claude Code) executa:**
1. **Ao mexer em código:** `/memory-ingest [arquivo ou documento]` → Documenta mudanças
2. **Ao completar feature:** `/memory-ingest feature-xyz.md` → Registra aprendizado
3. **Semanalmente (segunda 9h):** `/memory-consolidate` → Auditoria + síntese (automático com task)
4. **Ao precisar context:** `/memory-query [pergunta]` → Busca viva na wiki

**Você controla:**
- Estrutura (pode reorganizar `wiki/` conforme arquitetura evolui)
- Tipos de página (concept, feature, decision, integration, etc)
- Cadência de ingests e consolidações

---

## 📋 ACOMPANHAMENTO VIA TASKS

Use `/memory-ingest` para cada grande mudança:

```markdown
Quando terminar uma task de código, dispare:
/memory-ingest feature-xyz-completed.md

Isto:
✅ Documenta o que foi feito
✅ Extrai decisões e learnings
✅ Atualiza wiki automaticamente
✅ Registra em log.md (rastreabilidade)
```

**Consolidação Automática:** Toda segunda-feira às 9h
```
/memory-consolidate
→ Correlaciona código (GitHub) + wiki
→ Detecta gaps
→ Gera relatório tipo "Prisma migrations"
```

---

## DIRETRIZES DE EXECUÇÃO

Você deve atuar como **Engenheiro Autônomo** para executar os Módulos de desenvolvimento do projeto. Ao iniciar qualquer trabalho, **leia rigorosamente** o documento `.md` associado antes de mexer em código.

Conforme você for completando as tarefas dos documentos, use seus comandos para "ticar" os checkboxes (`[x]`) nos arquivos dos planos refletindo seu progresso.

### ⭐ NOVO: Acompanhamento via Knowledge Base

**Ao INICIAR uma task de desenvolvimento:**
1. Leia o `plan.master.*.md` relevante
2. Entenda o contexto (arquitetura, decisões prévias)
3. Documente PRÉ-REQUISITOS em `knowledge-base/wiki/decisions/`

**Enquanto TRABALHA:**
1. Faça mudanças incrementais
2. Após mudanças significativas: `/memory-ingest seu-arquivo.md`
3. Isso documenta aprendizados em tempo real

**Ao TERMINAR:**
1. Marque `[x]` no plan.master.md
2. Rode `/memory-ingest feature-xyz-summary.md`
3. Wiki automaticamente sincronizado

**Toda SEGUNDA 9h:**
1. `/memory-consolidate` (automático)
2. Relatório em `wiki/migrations/YYYY-MM-DD.md`
3. Você revisa gaps e aprende com timeline

---

## 🎯 As 4 Operações da Knowledge Base

Você tem acesso a estas operações via skill `/memory-*`:

### 1. Ingest — Documentar Mudanças
```
/memory-ingest feature-oauth-serpro.md

Fluxo:
  1. Lê o documento
  2. Extrai decisões, learnings, trade-offs
  3. Cria página em wiki/decisions/ ou wiki/features/
  4. Atualiza index.md
  5. Apend log.md com timestamp
```

**Use quando:**
- Termina feature (documentar o aprendizado)
- Toma decisão arquitetural importante
- Descobre padrão ou problema

### 2. Query — Buscar Conhecimento
```
/memory-query Como funciona a integração Serpro?

Fluxo:
  1. Busca em wiki/integrations/serpro.md
  2. Busca em wiki/security/ (certificados, OAuth)
  3. Sintetiza resposta com contexto
  4. Oferece arquivar em wiki/outputs/
```

**Use quando:**
- Precisa relembrar arquitetura
- Quer entender decisão passada
- Está perdido em novo módulo

### 3. Lint — Auditar Saúde da Wiki
```
/memory-lint

Verifica:
  ❌ Links quebrados
  ❌ Páginas órfãs
  ❌ Documentação desatualizada
  ❌ Gaps em cobertura
```

**Use quando:**
- Código evoluiu e wiki ficou pra trás
- Quer garantir consistência

### 4. Consolidate — Auditoria Temporal Semanal
```
/memory-consolidate

(Executado automaticamente toda segunda 9h via task)

Fluxo:
  1. Lê git log (GitHub commits)
  2. Correlaciona com wiki/log.md
  3. Detecta conflitos (código ≠ documentação)
  4. Gera relatório tipo "Prisma migrations"
  5. Atualiza overview.md (síntese)

Output:
  📄 wiki/migrations/YYYY-MM-DD.md (relatório)
  📊 wiki/overview.md (reescrito)
  🔗 Todos os links da semana correlacionados
```

---

## 📋 Fluxo Prático — Exemplo Real

**Segunda-feira 10h:**
```
Você inicia MÓDULO 4: Integra Contador

1. Lê docs/plan.master.integra.md
2. Consulta /memory-query "Qual é o contexto Serpro?"
   → Wiki responde com histórico de decisões
3. Começa a codificar
```

**Quarta-feira 14h (após implementar multi-empresa):**
```
Você termina feature de Gestão Multi-Empresa

1. Escreve wiki/features/multi-empresa.md
2. Roda /memory-ingest wiki/features/multi-empresa.md
   → Wiki atualizada automaticamente
3. Marca [x] no plan.master.integra.md
4. Próxima task
```

**Sexta-feira 16h (após bugs encontrados):**
```
Você descobre que Certificado SSL vence e causou erro

1. Documenta em wiki/security/certificados.md
2. Roda /memory-ingest wiki/security/certificados.md
   → Registra decision + learnings
3. Marca [x] em plan.master.md
```

**Segunda-feira 9h (AUTOMÁTICO):**
```
/memory-consolidate (executa automaticamente)

Resultado:
  📄 wiki/migrations/2026-04-27.md

Contém:
  Timeline:
    - 2026-04-22: Feature multi-empresa implementada
    - 2026-04-24: Bug certificado descoberto
    - 2026-04-25: Security docs atualizados
    - 2026-04-26: GitHub commits revisados
  
  Correlações:
    ✅ Código + documentação sincronizados
    ⚠️ Timeout do Serpro não documentado
  
  Sugestões:
    - [ ] Documentar timeout strategy
    - [ ] Adicionar alertas de certificado vencendo
```

---

## ✅ MÓDULO 1: Arquitetura Frontend e API — CONCLUÍDO
**Documento Guia:** `docs/plan.master.frontend.md` (Status: Concluído ✅)

O frontend Next.js foi transformado em cliente leve (BFF pattern). Toda lógica pesada foi movida para o `bot-backend`. Libs legadas (`pg`, `ioredis`, `node-forge`) foram removidas do `package.json` principal. As rotas `src/app/api/` agora são proxies HTTP para o backend Express.

---

## ✅ MÓDULO 2: Fluxo do Bot e Auditoria Base — CONCLUÍDO
**Documento Guia:** `docs/plan.master.bot.md` (Status: Concluído ✅)

O Agente Apolo foi modularizado em `bot-backend/src/ai/agents/apolo/` (prompt.ts + 3 workflows). O catálogo de serviços foi substituído pelo `{{DYNAMIC_CONTEXT}}` do Knowledge Base (Redis/DB). A tool `consultar_pgmei_serpro` foi criada como Camada 1 restrita (PGMEI + PGFN). O fluxo de Senha GOV foi refinado para não ser cobrado no primeiro atendimento.

---

## ✅ MÓDULO 3: Segurança Serpro e Pacote DART — CONCLUÍDO
**Documento Guia:** `docs/plan.master.serpro.md` (Status: Concluído ✅)

Payload Serpro ajustado (v2.4). Consultas restritas a pós-Procuração confirmada. Pacote Dart baseline inspecionado. Diretório `/dart-packages/serpro_integra_contador/` inicializado.

---

## 🚀 MÓDULO 4: Plataforma Integra Contador (Fase Atual)
**Documento Guia:** `docs/plan.master.integra.md`
**Documento de Pesquisa:** `integra.md`

### Contexto
O bot já consome a API Serpro de forma pontual (single-tenant, invocação manual via tools do Apolo). O objetivo agora é transformar essa integração num **módulo de plataforma completo**, inspirado no Calima ERP, que permita gerenciar múltiplos clientes, automatizar processos via robôs agendados e dar visibilidade total ao contador no painel Admin.

### Resumo das Ações a serem tomadas:
1. **Gestão Multi-Empresa:** Criar as tabelas `integra_empresas` e `integra_config` no banco. Construir CRUD no `bot-backend` (`/routes/integra/`) e as telas no Admin.
2. **Robôs (Jobs BullMQ):** Criar workers agendados para PGMEI, PGDAS, CND e Caixa Postal. Configuração de dia/hora por robô no painel Admin.
3. **Dashboard Integra Contador:** Cards de resumo + gráficos de status de guias + alertas de certificados a vencer.
4. **Armazenamento de Resultados:** Persistir guias (DAS, DARF) como PDF no R2, histórico de declarações e mensagens da Caixa Postal.
5. Leia o documento guia e siga as Fases estipuladas. Marque os `[x]` ao terminar.

---

## 🔧 Inicializar Knowledge Base (Primeira Vez)

Se ainda não tiver criado `knowledge-base/`, execute:

```bash
cd D:\Códigos\Haylander\haylanderform

# Crie estrutura
mkdir knowledge-base\raw knowledge-base\wiki knowledge-base\git
mkdir knowledge-base\wiki\{architecture,features,integrations,security,workflows,decisions,outputs,migrations}

# Copie CLAUDE.md customizado (veja abaixo)
# Crie index.md, log.md, overview.md iniciais
```

**OU dispare:**
```
/memory-ingest docs/plan.master.frontend.md

Isto vai:
  ✅ Criar knowledge-base/ automaticamente
  ✅ Inicializar estrutura
  ✅ Registrar plan.master.frontend.md como primeira fonte
```

---

## 📖 CLAUDE.md Customizado para Knowledge-Base

Salve em `knowledge-base/CLAUDE.md`:

```markdown
# Haylanderform — Knowledge Base Schema

**Projeto:** Haylanderform (CRM/ERP + WhatsApp para MEIs)  
**Manutenção:** Claude Code (autonomously)  
**Sincronização:** GitHub + Weekly Consolidation

## Role

Você (Claude Code) mantém esta wiki viva. Conforme você desenvolve:
1. Documenta features em wiki/features/
2. Documenta decisões em wiki/decisions/
3. Documenta integrações em wiki/integrations/
4. Ingest semanal via /memory-ingest
5. Consolidate semanal (segunda 9h) automático

## Estrutura

wiki/
├── index.md              ← Master catalog
├── log.md                ← Append-only operational log
├── overview.md           ← Living architecture synopsis
├── architecture/         ← System design, patterns
├── features/             ← Features implemented
├── integrations/         ← Serpro, Caixa, Calima, etc
├── security/             ← Certificates, OAuth, secrets
├── workflows/            ← Business flows (Bot, Contador)
├── decisions/            ← ADRs and trade-offs
├── migrations/           ← Weekly consolidation reports
└── outputs/              ← Query archives

## Types of Pages

- `architecture`: System designs, C4 diagrams
- `feature`: Implemented feature + decisions
- `decision`: ADR (Architecture Decision Record)
- `integration`: Third-party integration details
- `security`: Certificates, OAuth, API keys handling
- `workflow`: Business process flows
- `migration`: Weekly consolidation report

## Rules

✅ Use relative links: [Title](architecture/frontend.md)
✅ Frontmatter YAML on every page
✅ Update index.md + log.md on every operation
✅ Never edit raw/ (read-only)
✅ Flag contradictions explicitly
✅ Sync with GitHub weekly (consolidate)
```

---

## 🎯 Regras de Sobrevivência no Código
- **Nunca apague o .pfx ou Pkcs12 antigo sem que os Secrets estejam operacionais e devidamente mapeados.**
- Como você (`claude`) tem a habilidade de abrir arquivos no terminal e alterá-los em lote, tome extremo cuidado na hora de mover os arquivos `/src/app/api/` que você precisará refatorar.
- A comunicação real-time (Sockets) já existe e conecta o bot-backend ao admin board. Tente não quebrar.

### Best Practices de Engenharia (Inspiradas no Claude Code Leak)
Para evitar geração de "spaghetti code" e se proteger de limitações de contexto da IA, siga estritamente estas diretrizes operacionais:
1. **Verificação rigorosa pós-edição:** Após modificar código, SEMPRE rode as validações do projeto (`tsc`, lints, testes) para confirmar que a edição não quebrou nada. Não assuma sucesso sem validar.
2. **Releitura antes da edição:** Nunca edite um código apenas confiando no cache/memória do contexto longo. SEMPRE obtenha e leia (via `view_file` ou equivalente) a versão mais recente do arquivo antes de aplicar refatorações.
3. **Arquivos e Funções Grandiosas:** Leituras excedendo 2.000 linhas sofrem truncamento. Para código extenso, leia em "chunks". Mais importante: não crie funções gigantes. Aplicar refactoring contínuo para quebrar qualquer função que ultrapasse 50 linhas em módulos menores. Evitar ao máximo múltiplos níveis de aninhamento (nesting).
4. **Prevenção de Truncamento:** Outputs e retornos de console muito extensos (>50 mil caracteres) são cortados silenciosamente. Não rode cat ou processos cujo stdout seja caótico e extenso.
5. **Vibe Coding Disciplinado:** Execute em incrementos minúsculos e seguros. Teste a cada passo. O planejamento (`plan.master.md`) serve para quebrar tarefas imensas. Não tome atalhos arquiteturais; priorize a legibilidade e a baixa complexidade ciclomática.

---

## 📚 Knowledge Base Best Practices

1. **Após cada feature completada:** Dispare `/memory-ingest feature-summary.md`
2. **Após decisão importante:** Crie `wiki/decisions/ADR-NNNN.md` e ingest
3. **Quando descobre bug/problema:** Documente em `wiki/workflows/` ou `wiki/security/`
4. **Semanalmente (segunda 9h):** `/memory-consolidate` roda automaticamente
5. **Se perder em contexto:** `/memory-query [sua pergunta]` busca na wiki

---

## 📋 Checklist — Setup Inicial

- [ ] Crie pasta `knowledge-base/` em `D:\Códigos\Haylander\haylanderform/`
- [ ] Crie subdirs: `raw/`, `wiki/`, `git/`
- [ ] Crie `wiki/` subdirs: `architecture/`, `features/`, `integrations/`, `security/`, `workflows/`, `decisions/`, `migrations/`, `outputs/`
- [ ] Copie este CLAUDE.md para `knowledge-base/CLAUDE.md`
- [ ] Crie `wiki/index.md` (vazio ou com índice inicial)
- [ ] Crie `wiki/log.md` (início vazio)
- [ ] Crie `wiki/overview.md` (síntese inicial)
- [ ] Abra em Obsidian: `knowledge-base/wiki/`
- [ ] Configure agendamento: `/schedule consolidate-weekly` (segunda 9h)
- [ ] Primeiro ingest: `/memory-ingest docs/plan.master.frontend.md`

---

## 🚀 Próximos Passos

1. **Agora:** Crie a pasta `knowledge-base/` com estrutura
2. **Hoje:** Ingest do `plan.master.frontend.md` (modulo 1 já concluído)
3. **Esta semana:** Continue ingests conforme trabalha em MÓDULO 4
4. **Próxima segunda 9h:** Primeira consolidação automática
5. **Observação:** Wiki cresce com o projeto. Reavalie structure conforme necessário.

---

## 📞 Resumo

Você agora tem:
✅ Knowledge base automática (método Karpathy)
✅ Acompanhamento via wiki/log.md (rastreabilidade)
✅ Consolidação semanal (GitHub + wiki correlacionado)
✅ System para documentar decisões (wiki/decisions/)
✅ Context buffer (quando perder, `/memory-query`)

Isto permite que você desenvolva com **documentação viva**, não documentação que fica desatualizada.

---
> Source: [volticscontent/haylanderform](https://github.com/volticscontent/haylanderform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
