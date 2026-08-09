## building-ai-agents-from-zero-to-production

> Orientações para agentes de codificação de IA (e colaboradores humanos) que trabalham neste repositório. Se você é um

# AGENTS.md

Orientações para agentes de codificação de IA (e colaboradores humanos) que trabalham neste repositório. Se você é um
agente automatizado fazendo alterações aqui, leia este arquivo primeiro e siga-o.

## O que é este repositório

**Building AI Agents from Zero to Production** é um curso de aprendizado da Microsoft. Ele ensina desenvolvedores
a projetar, construir, avaliar, implantar e operar agentes de IA no **Microsoft Foundry** usando o
**Microsoft Agent Framework (MAF)**. O conteúdo está organizado como uma sequência de lições, cada uma com um
`README.md` e exemplos executáveis em Python.

```
lesson-1-agent-design/            Use case + how to design effective agents
lesson-2-agent-development/       Build specialised agents with MAF (multiple runnable samples)
lesson-3-agent-evals/             Evaluations and observability
lesson-4-agentdeployment/         Deploy a hosted agent + OpenAI ChatKit front end
lesson-5-hosted-agents-production/ Hosted Agents vs Capability Hosts, BYO storage, governance
lesson-6-toolbox/                 Microsoft Toolbox: define + govern tools centrally
lesson-7-multi-agent-a2a/         Multi-agent orchestration over the A2A protocol
```

Documentação principal: `README.md` (comece aqui), `MIGRATION-GUIDE.md` (detalhes da migração do SDK), `CHANGELOG.md`.

## Regras de ouro

1. **Nunca comite segredos.** Apenas arquivos `*.env.example` são monitorados; arquivos `.env` reais são
   git-ignored. Não codifique endpoints, chaves, tokens ou strings de conexão em exemplos ou na documentação.
2. **Não mexa em `translations/` ou `translated_images/`.** Eles são gerados automaticamente por uma
   GitHub Action de tradução. Nunca os edite manualmente; faça alterações na fonte nos
   arquivos de lição do nível superior apenas.
3. **Nenhum modelo obsoleto.** Use **`gpt-5.1`** para chat/avaliação e **`gpt-5-codex`** para codificação.
Do **not** introduce `gpt-4o`, `gpt-4.1`, or any retired model, and do not use *GitHub Models*
(retiring July 30, 2026) — all models are served through Microsoft Foundry.
4. **Use a superfície atual do SDK.** Os exemplos têm como alvo `agent-framework` (fixado em `requirements.txt`)
    com `FoundryChatClient` e a **Responses API**. Não reintroduza os
    padrões `AzureAIClient` / `AzureAIAgentClient` / `AzureOpenAIChatClient`.
5. **Mantenha a terminologia atualizada**: *Microsoft Foundry* (not "Azure AI Foundry"), *Microsoft Agent
Framework*, *Hosted Agents*, *Capability Hosts*, *Microsoft Toolbox*, *MCP / Hosted MCP*, *A2A*.

## Configuração

```bash
python -m venv .venv
# Windows:  .venv\Scripts\Activate.ps1
# macOS/Linux:  source .venv/bin/activate
pip install -r requirements.txt

az login                     # os exemplos se autenticam com sua identidade de desenvolvedor
cp .env.example .env         # então preencha o endpoint do seu projeto Foundry + modelo
```

Requisitos: **Python 3.12+**, o **Azure CLI**, e acesso a um projeto **Microsoft Foundry**
com um modelo da série GPT-5 implantado. Cada README da lição lista seus próprios pré-requisitos e as variáveis de ambiente
que precisa (veja o `.env.example` no nível da lição quando presente).

## Executando exemplos

A maioria dos exemplos da lição 2 inicia uma **DevUI** local em uma porta dedicada (por exemplo 8090–8096); o servidor A2A
na lição 7 escuta na porta 9000. Verifique o docstring/README de cada exemplo para o comando exato
e a porta. Como os exemplos chamam endpoints do Foundry ao vivo, eles precisam de um `.env` válido e `az login`.

## Validando alterações

Não há suíte de testes unitários; a validação é estática + ao vivo:

- **Verificação estática (deve passar antes do commit):** compile em bytecode cada exemplo.
  ```bash
  python -m py_compile $(git ls-files '*.py')
  ```
  No Windows PowerShell:
  ```powershell
  git ls-files '*.py' | ForEach-Object { python -m py_compile $_ }
  ```
- **Links do Markdown:** o job CI `static` executa `markdown-link-check`
   (config: `.github/workflows/markdown-link-check-config.json`). Verifique se quaisquer novos links externos
   resolvem (HTTP 200).
- **Teste rápido:** `.github/workflows/smoke-test-hosted-agent.yml` executa a ação AI Smoke Test
   contra um agente hospedado implantado (`workflow_dispatch`, OIDC). A execução de agentes ao vivo requer acesso ao Azure.

O CI (`static` job) descobre automaticamente arquivos `.py`, então novos exemplos são cobertos sem editar o
workflow. Não comite código que falhe em `py_compile`.

## Convenções de commit

- Escreva commits focados com mensagens claras e no imperativo.
- Inclua o trailer de co-autor em commits assistidos por agente:
  ```
  Co-authored-by: Copilot App <223556219+Copilot@users.noreply.github.com>
  ```
- Não comite caches gerados, ambientes virtuais ou arquivos `.env` (todos git-ignored).

## Onde fazer alterações específicas

| Alteração | Local |
|--------|----------|
| Narrativa do curso / texto da lição | `lesson-*/README.md` (somente fonte — nunca `translations/`) |
| Código executável | `lesson-*/**.py`, `setup_vector_store.py` |
| Dependências | `requirements.txt` (mantenha as versões fixas) |
| Documentação de variáveis de ambiente | `.env.example`, `.env.example` por lição |
| CI / verificação estática | `.github/workflows/` |
| Habilidades do curso para assistentes de IA | `.github/skills/` |

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->

---
> Source: [microsoft/Building-AI-Agents-From-Zero-To-Production](https://github.com/microsoft/Building-AI-Agents-From-Zero-To-Production) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
