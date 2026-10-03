## workshop-jev

> Agente de vendas em LangGraph: responde perguntas sobre os dados de vendas de uma empresa fictícia escolhendo uma tool que lê uma view do Postgres. Cada node de decisão (guardrail, triagem, verificação) roda com um LLM, com o Jev ou com os dois, e um frontend React compara resposta, latência, tokens e custo. É o exercício de um workshop e uma ferramenta reutilizável de medição.

# JEV Jornada

Agente de vendas em LangGraph: responde perguntas sobre os dados de vendas de uma empresa fictícia escolhendo uma tool que lê uma view do Postgres. Cada node de decisão (guardrail, triagem, verificação) roda com um LLM, com o Jev ou com os dois, e um frontend React compara resposta, latência, tokens e custo. É o exercício de um workshop e uma ferramenta reutilizável de medição.

O quê e por quê: `docs/PRD.md`. O que implementar: `docs/specs/` (comece pelo `README.md` de lá). Só implemente a partir de uma SPEC com status `aceita`.

## Como rodar

```
cd backend && uv sync && uv run uvicorn app.main:app --reload     # http://localhost:8000
cd frontend && npm install && npm run dev                          # http://localhost:5173
```

`PROVIDER_MODE=replay` é o padrão de desenvolvimento e não precisa de banco. Rode com chaves reais só quando a tarefa pedir.

Banco de vendas (modo live e gravação do replay): `docker compose up -d --wait` na raiz, Postgres em `localhost:5433`. Os scripts de `db/init/` só rodam com o volume vazio: `docker compose down -v` para refazer.

## Como testar

```
cd backend && uv run pytest
cd backend && uv run pytest -m db                                 # com o banco no ar
cd backend && PROVIDER_MODE=replay uv run python -m app.cli run --limit 3   # contrato: sem chaves
pre-commit run --all-files                                         # da raiz: ruff, gitleaks, higiene
cd frontend && npm run lint && npm run typecheck && npm run build && npm run test
```

Toda SPEC concluída tem teste; PR sem teste não entra.

## Convenções

- Conventional Commits, validados pelo commitlint: `tipo(escopo): descrição`, descrição começando em minúscula. Tipos e escopos em `commitlint.config.cjs`.
- Uma branch por SPEC (`feat/spec-01-...`), squash merge, título do PR cita a SPEC.
- Nunca commit direto em `main`, nunca `git push --force`, nunca mover tags.
- Antes de codar uma SPEC, carregue as skills da tabela em `docs/specs/README.md`. Onde uma skill contradiz o PRD, o PRD vence (Python 3.12, React 18).

## Regras do domínio

- O mesmo `NodeSpec` alimenta os dois providers. A única exceção é `llm_prompt_style: native` (SPEC-12): o LLM recebe os system prompts de `backend/config/prompts/`, com a mesma saída estruturada. Não crie outros prompts por provider.
- Em modo `both`, só o provider primário segue no fluxo; os dois vão para `metrics`.
- O node `reply` é só LLM. O node `tool` é código: roda a tool escolhida pelo primário.
- O nome da view vem só do registro em `backend/app/tools.py`; nada que o modelo devolve vira SQL.
- Views sem parâmetro e sem `now()`: o replay grava um resultado por tool em `fixtures/replay/tools/`.
- Preços vêm de `config/pricing.yaml`, nunca hardcoded.
- Tokens vêm da resposta da API e latência do relógio monotônico, nunca estimados.

## Não tocar sem pedir

`data/golden_set.json`, `backend/fixtures/replay/`, `db/init/` (seed e views), `backend/config/pricing.yaml` e os limiares em `backend/config/graph.yaml`. Mudanças neles alteram o resultado da comparação: exigem commit `data:` ou `config:` explícito e revisão humana.

## Segredos

Nunca leia, imprima ou commite valores do `.env`. Se encontrar uma chave em código, pare e avise.

## Definição de pronto

Lint limpo, testes passando, CI verde, modo replay funcionando, e este arquivo e a SPEC atualizados se as interfaces mudaram.

---
> Source: [caio-moliveira/workshop-jev](https://github.com/caio-moliveira/workshop-jev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
