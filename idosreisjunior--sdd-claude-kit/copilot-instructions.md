## sdd-claude-kit

> Este repositório desenvolve o **SDD Claude Kit**, um framework de Spec-Driven Development para Claude Code. O projeto é construído usando o próprio método que ele propõe: **a especificação vem antes do código**.

# CLAUDE.md — SDD Claude Kit

Este repositório desenvolve o **SDD Claude Kit**, um framework de Spec-Driven Development para Claude Code. O projeto é construído usando o próprio método que ele propõe: **a especificação vem antes do código**.

## Contexto obrigatório

Antes de qualquer mudança estrutural, leia:

1. `PRD.md` — fonte da verdade do produto.
2. `.specs/project/constitution.md` — regras invioláveis do projeto.
3. `.specs/project/architecture.md` — arquitetura e limites de módulos.
4. `.specs/project/standards.md` — convenções de código e documentação.
5. A especificação ativa em `.specs/features/<id>/`.

Carregue **apenas o necessário** para a tarefa atual (princípio 7.6 do PRD). Não leia todas as specs de uma vez.

## Regras obrigatórias

- Não implemente funcionalidades que não estejam no PRD ou na spec ativa.
- Não invente requisitos. Se faltar informação, registre uma **hipótese** explícita (`> HIPÓTESE:`) ou uma questão pendente na spec.
- Trabalhe em **uma tarefa por vez**.
- Antes de implementar, verifique requisitos, dependências e critérios de aceite da tarefa.
- Crie ou atualize testes para toda regra implementada.
- Atualize o status da tarefa em `tasks.md` e `status.yaml`.
- Atualize `traceability.yaml` (requisito → cenário → tarefa → arquivo → teste).
- Execute lint, testes e build antes de concluir.
- Não altere arquivos fora do escopo da tarefa sem justificar.
- Ao encontrar uma decisão arquitetural não prevista, **interrompa** e proponha um ADR.
- Não marque uma tarefa como concluída enquanto existirem falhas de validação.

## Fluxo de trabalho

```
/sdd-kit:new → spec → clarify → design → tasks → approve → implement → verify → archive
```

O estado de cada mudança vive em `.specs/<tipo>/<id>/status.yaml`. Transições válidas estão em `.specs/project/architecture.md`.

## Estrutura

| Caminho | Conteúdo |
| --- | --- |
| `plugins/sdd-kit/` | O plugin Claude Code (skills, agents, hooks, scripts, templates, schemas) |
| `packages/cli/` | CLI opcional (Fase 5 — ainda não implementada) |
| `.specs/` | Especificações deste repositório |
| `docs/pt-BR/`, `docs/en/` | Documentação de usuário |
| `examples/` | Projetos de exemplo |
| `tests/` | Testes automatizados do framework |

## Convenções de identificadores

| Prefixo | Uso |
| --- | --- |
| `RF-NNN` | Requisito funcional do produto (PRD §10) |
| `RNF-NNN` | Requisito não funcional do produto (PRD §20) |
| `REQ-<ESCOPO>-NNN` | Requisito funcional de uma feature |
| `NFR-<ESCOPO>-NNN` | Requisito não funcional de uma feature |
| `SCN-<ESCOPO>-NNN` | Cenário de aceite |
| `TASK-<ESCOPO>-NNN` | Tarefa de implementação |
| `TEST-<ESCOPO>-NNN` | Teste esperado |
| `ADR-NNN` | Decisão arquitetural |

## Comandos de validação

Definidos em `.specs/config.yaml` (`validation.commands`). Enquanto o pacote Node não existir, esses comandos ainda não estão disponíveis — registre isso no relatório de verificação em vez de fingir que passaram.

## Idioma

Documentos e specs em **pt-BR**. Identificadores, nomes de arquivo, código e mensagens de commit em **inglês**. A tradução para `en` é entrega da Fase 6.

---
> Source: [idosreisjunior/sdd-claude-kit](https://github.com/idosreisjunior/sdd-claude-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
