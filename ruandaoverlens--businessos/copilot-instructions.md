## businessos

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# BusinessOS — Registro de Agentes (AGENTS.md)

Cada agente e um subagente/skill em `.claude/agents/<slug>.md`. `slug` vira
`last_edited_by: agent:<slug>` nos arquivos que ele propoe.

## Mecanismo (porta unica)
Todo agente le e escreve pelos CLIs, que embrulham `lib/content/repository.ts` e herdam
validacao, write_policy, deteccao de conflito e escrita atomica:
- ler: `pnpm agent:read --id <section/entity>` (JSON `{ frontmatter, body }`; pegue
  `revision`/`status`/`ai_context`) ou `pnpm agent:read --section <s>` (lista EntityMeta).
- escrever: `pnpm agent:write --id <id> --editor agent:<slug> --base-revision <n>
  [--summary ... --tags a,b --set k=v --body-file <tmp>]`.
NUNCA editar `content/` com fs cru / `echo` / `Edit`.

**Tenant (ADR 0001).** A porta unica e a mesma; o que resolve "de quem sao os dados" e
o contexto de execucao. No modo `supabase`, os CLIs rodam com `service_role` no tenant
do **admin** (o founder) via `withAdminContext` — contornam a RLS e agem sobre a copia
das entidades do admin. No modo `file` (padrao de dev/testes), operam direto sobre
`content/`, sem contexto. Em ambos os casos, as regras (validacao, write_policy,
conflito, `propose -> needs_review`) sao identicas, herdadas de `repository.ts`.

A coluna **invocar por** lista a intencao do founder; na UI, ela e disparada pelo botao
**"Pedir a IA"** do card/pagina da entidade (que roteia para o agente de alcada correto).

| slug | secao | le | escreve (propoe) | write_policy alvo | invocar por (botao "Pedir a IA") |
|---|---|---|---|---|---|
| founder-coach   | founder   | founder/*                                   | founder/objetivo             | propose      | "articular objetivo" |
| market-map      | direcao   | direcao/mapa-do-mercado, ima-de-problemas   | direcao/mapa-do-mercado      | propose      | "mapear mercado" |
| problem-magnet  | direcao   | direcao/ima-de-problemas, founder/*         | direcao/ima-de-problemas     | propose      | "listar problemas" |
| icp             | direcao   | direcao/perfil-ideal-de-cliente + related   | direcao/perfil-ideal-de-cliente | propose  | "definir ICP" |
| value-thesis    | direcao   | direcao/tese-de-valor + related             | direcao/tese-de-valor        | propose      | "propor tese de valor" |
| offer-strategist| direcao   | direcao/oferta, tese-de-valor               | direcao/oferta               | propose      | "montar oferta" |
| validation-synth| validacao | validacao/*, direcao/perfil-ideal-de-cliente| validacao/oferta, primeiros-clientes | propose | "sintetizar validacao" |
| cash-flow       | caixa     | caixa/fluxo-de-caixa                         | caixa/fluxo-de-caixa         | propose      | "resumir caixa/runway" |
| seed-assistant  | (todas)   | entidade-alvo                               | qualquer status:empty        | propose      | "preencher rascunho" |
| context-linter  | (todas)   | content/**                                  | — (so relata)                | read-only    | "validar contexto" |
| summarizer      | (todas)   | entidade-alvo                               | patch em summary             | propose      | "atualizar resumo" |

## Convencoes
- founder_only (estilo-de-vida, erp): NENHUM agente escreve; so leem.
- Oferta duplicada: offer-strategist -> direcao/oferta; validation-synth -> validacao/oferta.
  Sem sync automatico entre os dois.
- Todo agente respeita os guardrails de docs/05-agent-integration.md secao 5.

---
> Source: [ruandaoverlens/BusinessOS](https://github.com/ruandaoverlens/BusinessOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
