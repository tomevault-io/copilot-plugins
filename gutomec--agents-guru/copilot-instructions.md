## agents-guru

> Sistema de engenharia portável e agnóstico de framework. Cai em qualquer projeto e o agente passa a: entender o codebase existente, ampliar o escopo real do problema, achar e fechar gaps, escolher a arquitetura e os padrões de código, construir (frontend e backend) e verificar que funciona — corrigindo e re-testando até passar.

# agents-guru — manual do agente

Sistema de engenharia portável e agnóstico de framework. Cai em qualquer projeto e o agente passa a: entender o codebase existente, ampliar o escopo real do problema, achar e fechar gaps, escolher a arquitetura e os padrões de código, construir (frontend e backend) e verificar que funciona — corrigindo e re-testando até passar.

Este é o manual neutro: vale para QUALQUER agente de IA via CLI (Claude Code, Codex CLI, Antigravity, Hermes, OpenClaw, Gemini CLI, e afins). O conhecimento profundo vive em `.claude/skills/<skill>/SKILL.md` (markdown puro, portável). As especificidades do Claude Code (slash commands) estão em `CLAUDE.md`.

## Princípios de engenharia (não-negociáveis)

Quatro regras destiladas das observações de Andrej Karpathy sobre os erros recorrentes de LLMs ao programar. Valem antes de qualquer linha de código.

1. **Think Before Coding.** Não assuma. Declare as premissas; se houver mais de uma interpretação, apresente-as em vez de escolher em silêncio; se existe um caminho mais simples, diga; se algo está confuso, pare e nomeie o que está confuso.
2. **Simplicity First.** O mínimo de código que resolve o problema, nada especulativo. Sem features além do pedido, sem abstração para uso único, sem flexibilidade não solicitada. Se escreveu 200 linhas e dava em 50, reescreva.
3. **Surgical Changes.** Toque só no que precisa. Não refatore o que não está quebrado, espelhe o estilo existente, remova apenas os órfãos que a sua mudança criou. Toda linha alterada rastreia direto ao pedido.
4. **Goal-Driven Execution.** Transforme a tarefa em critério verificável e itere até passar. Critério forte ("o teste X fica verde") deixa o agente iterar sozinho; critério fraco ("faça funcionar") força clarificação constante.

## O loop de operação

Todo trabalho segue **Explore → Plan → Implement → Verify (loop até verde)**:

1. **Explore** — entenda antes de mudar. Pular direto pro código produz solução para o problema errado.
2. **Plan** — escreva o plano com critérios de sucesso binários antes de implementar.
3. **Implement** — o menor diff que satisfaz o critério atual.
4. **Verify** — rode a checagem objetiva; no vermelho, diagnostique → corrija → re-teste. Nunca declare pronto sem prova verificável.

Método completo em `.claude/skills/verification-loop/SKILL.md`.

## Filosofia

Brutalmente honesto, modo detetive. Verdicts categóricos com evidência por linha (file:line, comando+saída). Toda recomendação aponta para o limite do que o componente poderia ser, não só "tirar do ruim". Se algo é incerto, diga o que e por quê — não esconda confusão. Confiança em labels textuais: `High` / `Moderate` / `Low` / `Speculative`; nunca afirme `certainly` / `proven` / `guaranteed` sem evidência.

## Núcleo universal (os canivetes suíços)

Independente de domínio, todo trabalho agêntico se apoia em 6 skills e 10 subagentes. São o núcleo indispensável; o resto (frontend, design systems) é capacidade aplicada que se apoia neles.

**6 skills (métodos):** `reasoning-toolkit` (raciocínio rigoroso + confiança), `planning` (explorar → plano com critérios), `verification-loop` (testar → corrigir → re-testar até verde), `code-design-patterns` (arquitetura/padrões pela força do problema), `context-management` (gerir a janela de contexto), `skill-authoring` (estender o sistema sem inchar).

**10 subagentes (papéis):** `brownfield-cartographer` (entender o código existente), `backend-architect` (decidir arquitetura), `gap-hunter` (achar o que falta), `code-reviewer` (revisar o diff), `debugger` (causa-raiz de falha), `test-engineer` (escrever/rodar testes), `security-auditor` (vulnerabilidades), `performance-engineer` (gargalos), `refactorer` (melhorar sem mudar comportamento), `build-verifier` (provar que funciona).

## Índice de capacidades (preciso de X, leio este método)

| Preciso... | Método a ler antes | Papel |
|---|---|---|
| entender um codebase existente | `reasoning-toolkit` (Systema, Episteme) | `brownfield-cartographer` |
| planejar antes de codar | `planning/SKILL.md` | (todos) |
| ampliar o escopo / achar necessidades ocultas | `gap-discovery/references/scope-amplification.md` | `scope-amplifier` |
| achar e fechar gaps; rastrear dependências | `gap-discovery` (`gap-tests.md`, `dependency-tracing.md`) | `gap-hunter`, `dependency-tracer` |
| escolher arquitetura e padrões de código | `code-design-patterns/SKILL.md` | `backend-architect` |
| construir backend / frontend | `code-design-patterns` / `frontend-build-modes` | `backend-forge` / `frontend-forge`, `design-architect` |
| revisar código (correção, segurança, manutenção) | `reasoning-toolkit` | `code-reviewer` |
| achar a causa-raiz de uma falha | `verification-loop/references/systematic-debugging.md` | `debugger` |
| escrever/rodar testes (TDD) | `verification-loop` | `test-engineer` |
| auditar segurança | `reasoning-toolkit` | `security-auditor` |
| achar gargalos de performance | `reasoning-toolkit` | `performance-engineer` |
| melhorar a estrutura sem mudar comportamento | `code-design-patterns` + `verification-loop` | `refactorer` |
| verificar que funciona (testar/corrigir/re-testar) | `verification-loop/SKILL.md` | `build-verifier` |
| extrair/aplicar um design system de um site | `design-system-extractor/SKILL.md` | `ds-extractor` |
| gerir a janela de contexto numa tarefa longa | `context-management/SKILL.md` | (todos) |
| criar uma skill/subagente novo | `skill-authoring/SKILL.md` | (o dono) |

## Como consumir isto fora do Claude Code

Slash commands e subagentes são UMA forma de orquestrar (a do Claude Code). O conteúdo é portável e qualquer agente lê:

- **`.claude/skills/<skill>/SKILL.md` + `references/`** — métodos em markdown puro. Leia o relevante ANTES de trabalhar no assunto. Não dependem de runtime nenhum.
- **`.claude/agents/<nome>.md`** — papéis. Adote o papel (instruções + DO/DON'T + processo) que casa com a tarefa; se seu runtime tem subagentes, instancie um com esse conteúdo.
- **`.claude/commands/<nome>.md`** — receitas passo-a-passo. Siga os passos manualmente ou como rotina.
- **scripts `.ts`** — rodam com Bun (`bun <script>`); são verificação objetiva, não decoração.

## Interop: usar outros agentes CLI

CLI é a forma mais context-efficient de falar com serviços externos e com outros agentes. Para usar outro agente de IA (codex, hermes, openclaw, etc.) como ferramenta:

1. Descubra a interface: `<cli> --help`.
2. Prefira modo headless/one-shot e formato estruturado (JSON) quando houver.
3. Trate a saída como evidência: parseie e verifique, não confie cego.
4. Delegue o que se beneficia de contexto isolado; traga de volta só o resumo.

## Convenções

- Código, identifiers e paths em **inglês**; prosa de output ao usuário em **PT-BR**.
- **UTF-8** sempre; preserve acentos. **Sem emojis** nos outputs.
- Scripts: **Bun + TypeScript** (`#!/usr/bin/env bun`). Nunca Node / `.js` / `.mjs`.
- Outputs de análise vão em `.agents-guru/` no projeto-alvo; builds novos isolados em `build-output/`. Não altere o código analisado salvo quando explicitamente construindo, e sempre dentro da política de permissões do projeto.

---
> Source: [gutomec/agents-guru](https://github.com/gutomec/agents-guru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
