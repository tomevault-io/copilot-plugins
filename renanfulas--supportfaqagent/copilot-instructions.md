## supportfaqagent

> Antes de editar qualquer coisa neste projeto, siga a skill `supportfaq-project-navigator`.

# supportFAQagent — Claude Code Instructions

Antes de editar qualquer coisa neste projeto, siga a skill `supportfaq-project-navigator`.
Antes de commitar, fazer push ou abrir PR, siga a skill `supportfaq-git-flow`.
Quando alguém perguntar o que fazer agora, siga a skill `supportfaq-next-step-planner`.

---

## Skill: supportfaq-project-navigator

---
name: supportfaq-project-navigator
description: Use when an AI agent needs to understand, navigate, plan, or modify the supportFAQagent project without guessing architecture.
---

### O que esta skill faz

Skill que ajuda o agente a navegar no projeto sem alucinar arquitetura. Ela orienta a leitura do `README.md`, `CONTRIBUTING.md` e dos docs certos para cada tipo de mudança, indicando pastas, responsabilidades, testes e riscos de atropelar outra frente.

Use this skill before changing project structure, code, docs, domains, integrations, or knowledge content.

Goal: understand the project from repository sources, read only the needed docs, and avoid hallucinating architecture.

Also preserve the product positioning in `docs/product-positioning.md`: commercial but technical, operationally safe, traceable, and honest about MVP limits.

### First Step

Always start with:

1. Read `README.md`.
2. Read `docs/product-positioning.md` when the task touches README, docs, PR text, public positioning, onboarding, or agent instructions.
3. Read `CONTRIBUTING.md`.
4. Identify the change area.
5. Read only the docs needed for that area.

Do not load every doc by default.

### Mapa de documentação (onde achar cada coisa)

Primeiro destino: `docs/project-map.md` (estado das frentes) e `docs/navigation.md` (roteador por tarefa). Docs organizados por pasta:

| Pasta | O que vive aqui |
| --- | --- |
| `docs/` (raiz) | Índices: `project-map.md`, `navigation.md`, `documentation-status.md`, `product-positioning.md`, `agent-skills.md`, `references-legacy.md` |
| `docs/architecture/` | Design, fronteiras, contratos e padrões |
| `docs/setup/` | Guias de instalação/configuração de ambiente |
| `docs/MVP/` | Planos técnicos majoritários do MVP |
| `docs/quality-plans/` | Planos detalhados por frente |
| `docs/runbooks/` | Procedimentos operacionais |
| `docs/security/` | Planos e contratos de segurança |
| `docs/archive/` | Concluído, substituído ou obsoleto |

Lookup por assunto (ex.): Hermes → `docs/architecture/integration-contracts.md` (contrato) + `docs/quality-plans/hermes-chat-bridge-plan.md` (plano) + `docs/runbooks/hermes-chat-cutover.md` (operação). Caminhos antigos de docs movidos: `docs/references-legacy.md`.

### Area Map

| Change area | Read before editing | Likely folders |
| --- | --- | --- |
| Product positioning, README, public docs, PR narrative, agent instructions | `docs/product-positioning.md`, `README.md`, `docs/agent-skills.md` | `README.md`, `docs/`, `.agents/skills/` |
| Architecture or module boundaries | `docs/architecture/architecture.md`, `docs/MVP/technical-implementation-plan.md`, `docs/navigation.md` | `app/`, `docs/` |
| API contracts | `docs/architecture/integration-contracts.md`, `docs/architecture/observability.md` | `app/api/routes/`, `app/api/schemas/`, `app/feedback/`, `tests/` |
| Domain behavior | `docs/architecture/domain-contract.md`, `docs/navigation.md` | `domains/<domain>/domain.yaml`, `app/domain_engine/` |
| Knowledge base or FAQs | `docs/architecture/knowledge-authoring.md`, `docs/architecture/domain-evals.md` | `domains/<domain>/knowledge/`, `domains/<domain>/evals/` |
| Evals or calibration | `docs/architecture/domain-evals.md`, `docs/architecture/knowledge-authoring.md` | `app/evals/`, `domains/<domain>/evals/`, `tests/` |
| Ingestion | `docs/architecture/integration-contracts.md`, `docs/MVP/technical-implementation-plan.md` | `app/ingestion/`, `app/api/routes/ingestion.py`, `app/api/schemas/ingestion.py` |
| GitHub document loader | `docs/architecture/knowledge-authoring.md`, `docs/MVP/technical-implementation-plan.md`, `docs/navigation.md` | `app/ingestion/github_loader.py`, `scripts/fetch_github_document.py`, `tests/test_github_loader.py` |
| Retrieval or vector store | `docs/architecture/architecture.md`, `docs/MVP/technical-implementation-plan.md` | `app/retrieval/`, `app/orchestration/` |
| LLM/provider/prompt | `docs/MVP/technical-implementation-plan.md`, `docs/architecture/domain-contract.md` | `app/llm/`, `app/orchestration/`, `domains/<domain>/prompts/` |
| Handoff or escalation | `docs/architecture/domain-contract.md`, `docs/architecture/integration-contracts.md`, `docs/architecture/domain-evals.md` | `app/handoff/`, `app/orchestration/`, `domains/<domain>/domain.yaml`, `tests/` |
| Observability/logging | `docs/architecture/observability.md`, `docs/MVP/technical-implementation-plan.md` | `app/core/`, `app/main.py`, route files |
| Security or public surface hardening | `SECURITY.md`, `docs/security/`, `docs/architecture/observability.md`, `docs/architecture/code-standards.md` | `app/core/`, `app/api/`, `tests/security/`, `.github/workflows/` |
| External transport/automation (Meta WhatsApp, Hermes) | `docs/architecture/integration-contracts.md`, `docs/architecture/observability.md`, `docs/MVP/technical-implementation-plan.md` | docs first; do not move intelligence into external transport |
| PostgreSQL/pgvector | `docs/MVP/technical-implementation-plan.md`, `docs/architecture/architecture.md`, `docs/runbooks/pgvector-promotion-checklist.md` | `app/db/`, `app/retrieval/`, `app/ingestion/pgvector_writer.py`, `migrations/`, `scripts/ingest_domain_pgvector.py` |
| VPS/deploy/runtime | `docs/setup/environments.md`, `docs/MVP/technical-implementation-plan.md`, `docs/architecture/observability.md`, `docs/runbooks/` | `scripts/`, config/docs |
| Dependency management or security audit | `pyproject.toml`, `CONTRIBUTING.md`, `.github/workflows/security.yml` | `pyproject.toml`, `.github/workflows/` |
| Local chat UI or static assets | `README.md`, `docs/setup/environments.md`, `docs/MVP/technical-implementation-plan.md` | `app/static/`, `app/main.py`, `app/core/config.py` |

### Ownership Boundaries

- Renan: architecture, orchestration, tests, security, contracts, quality, docs.
- Alexandre: PostgreSQL, pgvector, persistence, workflows.
- Juliano: LangChain utilities, splitter/loaders, RAG support without overcoupling.
- Silotto: VPS, deploy, runtime environment, networking, logs.

If a task touches another person's primary area, prefer creating a contract, doc, adapter, or test seam instead of implementing their full responsibility.

### Project Rules

- Keep the core reusable across domains.
- Put domain-specific behavior in `domains/`, not hardcoded in `app/`.
- Routes should validate and orchestrate, not hold business logic.
- Prefer small adapters and interfaces over heavy framework coupling.
- Keep external automation/transport outside the intelligence core.
- Treat PostgreSQL + pgvector as the planned production vector store.
- Treat Chroma as local/prototype unless the team explicitly decides otherwise.
- Treat `pyproject.toml` as the only dependency source of truth; do not create parallel dependency lists.
- Use the official GitHub Contents API for GitHub document ingestion; do not scrape GitHub HTML.
- Keep public communication commercial-technical: explain business value, traceability, safe fallback, and human handoff without promising full autonomy.
- Do not log raw PII, tokens, secrets, prompts with sensitive data, or raw `session_id`.

### Before Editing

Answer these internally:

1. Is this shared core behavior or domain-specific behavior?
2. Which docs define the current contract?
3. Which owner/frente could be affected?
4. What test or eval should prove the change?
5. Does this require documentation updates?

### Validation Guidance

- Any code change: `python -m pytest`
- Any Python module change: `python -m compileall app tests scripts`
- Domain, prompt, retrieval, handoff, or knowledge change: `python -m app.evals.run_domain_eval suporte-vps-whatsapp`
- API contract change: add/update endpoint tests and update `docs/architecture/integration-contracts.md`
- Knowledge article change: update eval references when expected behavior changes
- Dependency change: `python -m pip_audit .`, `python -m pip check`

---

## Skill: supportfaq-git-flow

---
name: supportfaq-git-flow
description: Use when an AI agent needs to prepare tests, review changes, commit, push, or write a pull request for the supportFAQagent project.
---

### O que esta skill faz

Skill que facilita testes, commit, push e descrição de PR. Ela orienta o agente a ler o fluxo de contribuição, revisar `git status` e `git diff`, confirmar o escopo das mudanças, rodar validações e montar um template de PR claro.

Use this skill before committing, pushing, or opening a PR.

### Before Commit

1. Read `CONTRIBUTING.md`.
2. Run `git status --short --branch`.
3. Run `git diff --stat`.
4. Confirm the change has one clear scope.
5. Check if docs or evals must be updated.
6. If the change touches public docs, README, PR narrative, or agent instructions, check `docs/product-positioning.md`.

Do not commit unrelated changes.

### Test First

```bash
python -m pytest
python -m compileall app tests scripts
```

Also run evals when the change touches domain behavior, knowledge base, prompt building, retrieval, handoff, or LLM response behavior:

```bash
python -m app.evals.run_domain_eval suporte-vps-whatsapp
```

Dependency/security checks when `pyproject.toml` changes:

```bash
python -m pip check
python -m pip_audit .
```

### Commit Style

- Use imperative mood, short and specific.
- Mention the intent, not every file.
- Examples: `Add ingestion preview contract`, `Harden request observability privacy`, `Improve support knowledge baseline`
- Avoid: `fix`, `updates`, `misc changes`, mixing unrelated changes.

### Push

```bash
git push -u origin <branch-name>
```

Branch prefix: `codex/<short-topic>`

### PR Description Template

```md
## O que foi feito

Resumo curto da entrega e por que ela existe.

## Principais mudanças

- Mudança principal 1.
- Mudança principal 2.

## Validação

- `python -m pytest`
- `python -m compileall app tests`
- `python -m app.evals.run_domain_eval suporte-vps-whatsapp` quando aplicável
```

### Atualizar o estado dos planos (antes de finalizar)

Ao terminar uma fase/frente, antes do commit/PR, atualize a documentação de estado:

1. Plano da frente em `docs/quality-plans/<frente>.md`: o que foi entregue e o que falta (plano encerrado vai para `docs/archive/`).
2. `docs/project-map.md`: status da frente (✅ feito / 🟡 em andamento / ⬜ falta).
3. `docs/documentation-status.md`: só quando mudar status, ownership, contrato, migration ou operação.

Registre concreto, ex.: "Frente handoff: Fase 2 concluída; implementado ticket no support inbox e notificação WhatsApp; falta enriquecimento de push". A guarda `tests/test_docs_links.py` falha se um link de doc quebrar.

---

## Skill: supportfaq-next-step-planner

---
name: supportfaq-next-step-planner
description: Use when someone asks what to do next, wants to start a task, or needs to validate alignment with the technical plan without overlapping another contributor.
---

### O que esta skill faz

Skill que ajuda a decidir "o que fazer agora" sem atropelar o plano técnico ou outra frente do time. Verifica `main`, últimos commits/PRs e docs do plano, pergunta o que a pessoa pretende mexer e qual o responsável, e sugere o menor próximo passo seguro.

### Core Rule

Ask only these two questions when the user's intent or identity is missing:

```
O que você pretende implementar ou mexer?
Qual seu nome/responsável pela frente?
```

If both are already clear from the conversation, do not ask again.

### Required Context Check

```bash
git status --short --branch
git log --oneline --decorate -15
```

### Responsibility Map

| Pessoa | Responsabilidade primária |
| --- | --- |
| Silotto - TekZoom HG | VPS, deploy, ambiente, networking, logs, operação de runtime |
| Alexandre Madeira | PostgreSQL, pgvector, persistência, workflows, integrações |
| Juliano Barreto | LangChain utilities, splitter, loaders, suporte RAG sem acoplamento excessivo |
| Renan | Arquitetura, orquestração, testes, segurança, contratos, qualidade, docs, coordenação |

### Planning Output Format

```md
## Status atual
Resumo curto do que a main já tem relacionado à essa área.

## Alinhamento com o plano
Se está dentro do plano, adiantado demais, bloqueado ou dependente de outra frente.

## Risco de atropelo
Baixo/médio/alto e por quê.

## Próximo passo seguro
Menor tarefa implementável agora.

## Docs e arquivos para ler
Lista curta.

## Validação esperada
Testes, evals ou checks que devem rodar.
```

### Decision Heuristics

- Contracts, tests, docs, evals, safety, orchestration → Renan can usually proceed.
- Tables, migrations, pgvector queries, persistence, workflow storage → involve Alexandre.
- LangChain dependency strategy, splitter behavior, loaders → involve Juliano.
- Deploy, VPS logs, TLS, reverse proxy, environment variables, runtime networking → involve Silotto.
- Public exposure, security, secrets, PII, logs → require hardening and tests.

### What Not To Do

- Do not implement before asking intent and name when missing.
- Do not recommend rewriting the architecture from scratch.
- Do not move intelligence rules into external transport/automation.
- Do not make Chroma the production source of truth unless the team explicitly decides.
- Do not bypass pgvector ownership by creating a parallel production vector store.

---
> Source: [renanfulas/supportFAQagent](https://github.com/renanfulas/supportFAQagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
