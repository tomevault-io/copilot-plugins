## portal-do-arinos-sistema

> Nenhum agente trabalha sem contexto, nenhuma task termina sem memoria, e nenhuma memoria fica presa a um unico agente.

# GG.AI Labs SynkOS AGENTS.md - Portal do Arinos

## Regra de ouro

Nenhum agente trabalha sem contexto, nenhuma task termina sem memoria, e nenhuma memoria fica presa a um unico agente.

## Antes de qualquer task

1. Ler `docs/architecture.md`
2. Ler `docs/prd.md`
3. Ler `docs/backlog.md`
4. Ler `docs/stories/index.md`
5. Ler a story ativa em `docs/stories/{id}.md`

## Ao concluir qualquer task

1. Atualizar `File List` da story
2. Gerar `session_handoff` em `.synko/vault/projects/synko/sessions/`
   com nome `session-{YYYY-MM-DD}-{story-id}.md`
3. Respeitar o `gate_profile` da story ao validar a execucao:
   `code` -> lint, typecheck, test
   `infra` -> smoke test, rollback note
   `docs` -> revisao de conteudo
   `discovery` -> findings documentados
4. Marcar acceptance criteria cumpridos
5. Criar ADR se houver decisao arquitetural

## Principio operacional

`MCP/Runtime First -> Observability Second -> UI Third`

## Regras de execucao

- Nao crescer escopo no meio da story
- Gaps novos viram novo backlog item
- `blocked`, `paused` e `failed` devem ser tratados de forma distinta
- Se a task tocar runtime existente, fazer brownfield discovery antes de alterar

## Referencias

- `_references/sources.md`
- `docs/backlog.md`
- `docs/stories/index.md`

---
> Source: [guilhermegiorgi/portal-do-arinos-sistema](https://github.com/guilhermegiorgi/portal-do-arinos-sistema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
