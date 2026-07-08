## auraxis-platform

> > Ponto de entrada para agentes AI que trabalham no nível de platform/orquestração.

# AGENTS.md — auraxis-platform

> Ponto de entrada para agentes AI que trabalham no nível de platform/orquestração.
> Para Claude Code, o arquivo canônico é `CLAUDE.md`.

## Identidade

Repositório orquestrador do Auraxis. Contém governança, contexto compartilhado e automações
para os repos de produto: auraxis-api, auraxis-web, auraxis-app.

## Regra de ouro

**Não escreva código de produto aqui.** Para features, vá ao repo correto em `repos/`.

## Passo 1 — Verificar coordenação OBRIGATÓRIO

```bash
cat .context/active_agents.json
```

Este arquivo é a fonte de verdade sobre quem está trabalhando no quê em TODOS os repos.
Se o repo/issue que você quer está ocupado: aguarde ou escolha outra.

## Passo 2 — Registrar trabalho

```bash
# Atualizar sua entrada antes de começar:
# {
#   "agent": "codex",
#   "issue": 123,
#   "repo": "auraxis-web",
#   "branch": "feat/codex-my-feature",
#   "since": "2026-05-10T10:00:00Z"
# }
```

## Repos disponíveis e seus AGENTS.md

| Repo | AGENTS.md | Stack |
|------|-----------|-------|
| `repos/auraxis-api` | `repos/auraxis-api/AGENTS.md` | Python · Flask · PostgreSQL |
| `repos/auraxis-web` | `repos/auraxis-web/AGENTS.md` | Nuxt 4 · Vue 3 · TypeScript |
| `repos/auraxis-app` | `repos/auraxis-app/AGENTS.md` | React Native · Expo · TypeScript |

## Convenção de branch

```
feat/<issue>-<desc>
fix/<issue>-<desc>
docs/<issue>-<desc>
chore/<issue>-<desc>
```

## Entrega obrigatória

- Todo trabalho concluído deve ter branch própria, issue vinculada, card no GitHub Projects
  e PR aberto com o escopo completo.
- O PR final deve ser aberto como pronto para revisão, **não em draft**, salvo pedido explícito
  do humano em contrário.
- O corpo do PR deve conter resumo, validação executada, riscos residuais e `Closes #<issue>`.
- Para mudanças web ou mobile com impacto visual, anexar screenshots/imagens na descrição do PR
  sempre que viável. Se não for viável, explicar o motivo no PR.

## Scripts úteis

```bash
# Health check de toda a platform
bash scripts/check-health.sh

# Sincronizar issues com GitHub Projects
GITHUB_TOKEN=$(gh auth token) python3 scripts/sync_repo_issues_to_project.py

# Ver snapshot do board
GITHUB_TOKEN=$(gh auth token) python3 scripts/sync_github_projects.py --mode pull

# Limpar worktrees antigos (branches mergeadas)
bash scripts/cleanup-worktrees.sh --dry-run

# Verificar staleness do OpenAPI snapshot
bash scripts/check-openapi-staleness.sh
```

## O que pode fazer aqui

- Ler/atualizar `.context/` (exceto arquivos de governança — pedir permissão)
- Criar/atualizar `docs/wiki/`
- Criar issues e cards no GitHub Projects
- Rodar scripts de sincronização
- Atualizar `AGENTS.md` e `CLAUDE.md`

## O que NUNCA fazer

- `git add .` — sempre seletivo
- Push para `master` sem PR
- Modificar `.context/07_steering_global.md` ou `.context/08_agent_contract.md` sem aprovação humana
- Alterar scripts de CI/CD sem aprovação humana

---
> Source: [italofelipe/auraxis-platform](https://github.com/italofelipe/auraxis-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
