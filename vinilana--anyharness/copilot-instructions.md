## anyharness

> Servidor MCP que expõe o **harness inteiro** (Guides + Sensors + Judges + Contracts + Sessions) como protocolo único, agnóstico ao agente. Pacote npm: `anyharness`. Binário: `anyharness`. Funciona em qualquer cliente MCP (Claude Code, Cursor, Codex). Complementar a `@dotcontext/cli` (feedforward em `.context/`); este projeto entrega o feedback + execução em `.harness/`.

# anyharness

Servidor MCP que expõe o **harness inteiro** (Guides + Sensors + Judges + Contracts + Sessions) como protocolo único, agnóstico ao agente. Pacote npm: `anyharness`. Binário: `anyharness`. Funciona em qualquer cliente MCP (Claude Code, Cursor, Codex). Complementar a `@dotcontext/cli` (feedforward em `.context/`); este projeto entrega o feedback + execução em `.harness/`.

**Spec completa:** `SPEC.md` (sempre source-of-truth — divergências resolvem-se a favor da spec).

## Estrutura do repositório (single TS package)

```
src/
├── server/              # MCP server (stdio), registro de tools/resources/prompts
├── sensors/             # sensor.list/run/register + adapters (eslint, tsc, semgrep, ArchUnit)
├── judges/              # judge.list/review/record + rubric loader + schema validator (sem LLM caller)
├── contracts/           # specs + tasks (behaviour harness)
├── sessions/            # session.start/append/get + jsonl persistence
├── steering/            # harness.steer.suggest + log consumer
├── storage/             # .harness/ filesystem layout, read/write helpers
├── prompts/             # workflow.PREVC, workflow.bug-fix
└── shared/              # zod schemas, output normalizer, types
.harness/                # exemplo / template instalável
├── guides/{rules,skills,subagents}/
├── sensors/<id>.yaml
├── judges/<rubric>.{md,schema.json}
├── contracts/{specs,tasks}/
├── workflows/<id>.md
└── .local/              # sessions/, steering/log.jsonl (gitignored)
tests/                   # vitest, integration tests com cliente MCP
```

## Time de agentes (`.claude/agents/`)

- **harness-architect** — coerência cross-cutting, decisões, alinhamento à spec e ao posicionamento `@dotcontext/*`.
- **harness-mcp-server** — servidor stdio, registry de tools/resources/prompts, plumbing.
- **harness-sensors** — sensor adapters, output normalization, registry, regulation tagging.
- **harness-judges** — judges, rubrics, prompt versioning, schema validation, fixture suite. Servidor não chama LLM — apenas renderiza o prompt para o cliente MCP avaliar via `judge.review` → `judge.record`.
- **harness-contracts** — Specs + Tasks (behaviour harness — bloco mais não-trivial).
- **harness-sessions** — sessions API, jsonl event log, steering log, `harness.steer.suggest`.
- **harness-test-engineer** — integration tests com cliente MCP real, fixtures, e2e em Claude Code/Cursor.

## Convenções de desenvolvimento

- **Linguagem:** TypeScript estrito, Node 20+. Sem `any` em código novo.
- **Schemas first.** Todo input/output de tool e todo formato persistido em `.harness/` passa por zod. Schemas em `src/shared/schemas/`.
- **Output normalizado.** Sensors e judges retornam o JSON único definido em SPEC §9 (`tool, regulation, passed, summary, inconclusive, violations[]`). Stdout cru fica como fallback de passthrough.
- **Regulation obrigatória.** Toda sensor/judge declara `maintainability | fitness | behaviour`. Permite filtrar e cruzar coverage.
- **`.harness/` é source-of-truth do harness.** Servidor lê em runtime e observa via watcher. Tools como `sensor.register` escrevem ali (única write path).
- **Local-first.** `.harness/.local/` (sessions, steering log, telemetry) gitignored. Resto comitado.
- **MCP separation.** Guides → resources (`harness://guides/...`). Sensors/judges/contracts/sessions/steering → tools. Workflows → prompts.
- **Sem lock-in de cliente.** Funciona em Claude Code, Cursor, Codex. Testes integrados rodam contra cliente MCP real.
- **Judges não chamam LLM no servidor.** `judge.review` devolve prompt + schema + contexto; o cliente MCP conectado faz a inferência; `judge.record` valida o output da review pelo schema da rubric. Determinismo é responsabilidade do cliente; o servidor garante validação e versionamento do prompt em `.harness/judges/<rubric>.md`.
- **Idempotência onde faz sentido.** `sensor.register` com mesmo id+command é no-op. `session.start` é único por chamada (gera novo id). `contract.task.complete` falha se task já estiver completa.

## Princípios para o agente trabalhando no Harness MCP

1. **Leia a spec antes de criar arquivo novo.** Cada decisão mapeia pra uma seção da `SPEC.md`.
2. **Não invente escopo.** Itens em §13 (fora do escopo) precisam de confirmação explícita do usuário.
3. **Behaviour harness = Contracts + Judges.** Esses dois blocos têm que conversar: `contract.spec.validate` roda os sensor checks in-line e devolve os judge checks como `pending: requires_agent_review` para o cliente resolver via `judge.review` → `judge.record`.
4. **Steering loop é o entregável estratégico.** Sensors/judges geram log → `steer.suggest` propõe novos guides → humano aprova. Sem esse ciclo fechado, o produto vira lib genérica.
5. **Generated files** (se houver) levam header `<!-- Generated by anyharness — edit .harness/ instead. -->`.
6. **Erros bloqueantes vs informativos.** Sensor que falha por bug do código → `passed: false`. Sensor que falha por infra (timeout, rate limit, command not found) → `inconclusive: true`.

## Decisões fechadas (SPEC §15)

- **Pacote:** `anyharness` (npm + binary).
- **Tool namespace:** sem prefixo (`sensor.list`, `judge.review`, `judge.record`); exceção `harness.steer.suggest`.
- **URI scheme:** `harness://`.
- **Sessions/steering log:** gitignored em `.harness/.local/`.
- **Workflows:** primariamente `prompts/` MCP, com fallback resource `harness://workflows/<id>`.
- **Judges sem LLM no servidor.** Sem `ANTHROPIC_API_KEY`, sem default model, sem dependência de `@anthropic-ai/sdk`. Cliente MCP executa a inferência.

---
> Source: [vinilana/anyharness](https://github.com/vinilana/anyharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
