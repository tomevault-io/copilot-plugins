## lake-house-lab

> Quadro de agentes Claude Code especializados para o projeto. Cada agente tem missão, escopo de arquivos e dependências explícitas.

# AGENTS.md — LakeHouse Lab

Quadro de agentes Claude Code especializados para o projeto. Cada agente tem missão, escopo de arquivos e dependências explícitas.

## Regras Gerais

1. **Contratos antes de implementação** — nunca implemente sem que o schema/endpoint/interface esteja definido
2. **Sem refactor por preferência** — só refatore se há bug ou acoplamento que bloqueia evolução
3. **Linguagem dos documentos** — PT-BR para docstrings e comentários; código em inglês
4. **Testes antes de marcar como pronto** — rodar `pytest` e testar endpoints relevantes
5. **PRs por módulo** — uma PR por funcionalidade, nunca um commit gigante com tudo

---

## Agentes Disponíveis

### lakehouse-architect
**Missão:** Diagnosticar bugs de backend, implementar novas features Django/DRF/DuckDB e fazer análise de schema de dados.

**Diretórios PERMITIDOS:**
- `backend/lakehouse/` (todos os .py)
- `backend/config/`
- `backend/tests/`
- `backend/dbt_project/`

**Diretórios PROIBIDOS:**
- `frontend/src/` (não modifica componentes React)
- `.claude/agents/` (não modifica definições de agentes)

**Quando usar:** Bugs de SQL/DuckDB, novos endpoints DRF, modelos Django, tasks Celery, dbt models.

---

### lakehouse-code-reviewer
**Missão:** Analisar o estado atual do projeto, identificar gaps, dívidas técnicas e produzir relatório estruturado de status.

**Diretórios PERMITIDOS:** Todo o repositório (somente leitura — não modifica arquivos)

**Quando usar:** Antes de iniciar uma fase de desenvolvimento; quando precisar entender o estado atual do projeto.

---

### lakehouse-frontend-engineer
**Missão:** Implementar e melhorar componentes React, hooks, UX e integração com a API do backend.

**Diretórios PERMITIDOS:**
- `frontend/src/` (todos os arquivos)
- `frontend/package.json`
- `frontend/vite.config.js`

**Diretórios PROIBIDOS:**
- `backend/` (não modifica Python)
- `docker-compose.yml`
- `backend/dbt_project/`

**Contratos que NÃO PODE mudar:**
- Design system: cores (`--accent`, `--bg`, `--surface`, `--fg`), fontes
- Shape dos objetos retornados pela API — consume apenas campos documentados

**Quando usar:** Novos componentes, melhorias de UX, integração com novos endpoints, bugs de frontend.

---

### lakehouse-test-engineer
**Missão:** Escrever e manter testes — pytest para backend, Vitest para frontend — e configurar CI/CD.

**Diretórios PERMITIDOS:**
- `backend/tests/` (adicionar fixtures e novos testes)
- `frontend/src/**/*.test.jsx` (novos arquivos de teste)
- `frontend/vitest.config.js`
- `frontend/package.json` (devDependencies apenas)
- `.github/workflows/`

**Diretórios PROIBIDOS:**
- `backend/lakehouse/` (não modifica código de aplicação)
- `frontend/src/components/*.jsx` (não modifica componentes — adapta os testes)
- `frontend/src/hooks/useApi.js` (apenas mock em testes)

**Regra crítica:** Nunca mocke o DuckDB em testes de integração — use conexão real em arquivo temporário.

**Quando usar:** Adicionar cobertura de testes, configurar Vitest, CI/CD.

---

### tech-lead-orchestrator
**Missão:** Análise estratégica do repositório, gap analysis, roadmap em ondas e geração de handoffs para outros agentes.

**Diretórios PERMITIDOS:** Todo o repositório (leitura); cria apenas arquivos `.md` de documentação

**Diretórios PROIBIDOS:** Qualquer arquivo `.py`, `.jsx`, `.yml` de configuração de aplicação

**Quando usar:** Início de uma nova fase de desenvolvimento; quando precisar de um plano estruturado com paralelismo de agentes.

---

## Matriz de Paralelismo

| Agente A | Agente B | Relação | Motivo |
|---|---|---|---|
| lakehouse-architect | lakehouse-frontend-engineer | PARALELO SEGURO | Paths completamente disjuntos |
| lakehouse-architect | lakehouse-test-engineer | PARALELO SEGURO | Test engineer só adiciona em `tests/` |
| lakehouse-frontend-engineer | lakehouse-test-engineer | SERIALIZADO | Testes após componentes estáveis |
| tech-lead-orchestrator | qualquer outro | PARALELO SEGURO | Só lê e cria `.md` |

---

## Atualização deste arquivo

Atualizar sempre que:
- Um novo agente for criado em `.claude/agents/`
- As restrições de path de um agente mudarem
- Uma nova onda do roadmap for concluída e as dependências entre agentes evoluírem

---
> Source: [pizanao/lake-house-lab](https://github.com/pizanao/lake-house-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
