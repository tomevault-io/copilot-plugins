## harness-dio

> O `AGENTS.md` da raiz é a fonte da verdade sobre stack, arquitetura, comandos e

# Instruções do repositório para o GitHub Copilot

O `AGENTS.md` da raiz é a fonte da verdade sobre stack, arquitetura, comandos e
regras inegociáveis. Leia-o primeiro. Aqui fica apenas como o Copilot deve operar.

## Antes de começar
1. Procure um teste que já cubra o comportamento que você vai mudar.
2. Se não houver, escreva o teste antes da implementação — ele precisa falhar primeiro.
3. Decisão de arquitetura ou de regra de negócio não é sua: pergunte.

## Ao responder
- Responda em português do Brasil.
- Mostre o diff, não o arquivo inteiro, quando a mudança for pontual.
- Termine com: arquivos alterados, saída de `make tudo`, o que ficou fora do escopo.
- Diga explicitamente qualquer premissa que você assumiu e que não estava no pedido.

## Ao abrir pull request
- Título no imperativo, até 72 caracteres.
- Descrição com: problema, solução, como testar, risco de rollback.
- Vincule a issue com `Closes #N`.
- PR que toca `app/infra` ou `migrations/` exige revisor humano nomeado.

## Limites
- Não execute comando que altere ambiente remoto (deploy, `alembic upgrade` fora
  do local, `az`, `gh pr merge`).
- Não altere arquivos em `.github/` sem pedido explícito.
- Ao encontrar segredo commitado, pare e reporte. Não tente remediar sozinho.

---
> Source: [digitalinnovationone/harness-dio](https://github.com/digitalinnovationone/harness-dio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
