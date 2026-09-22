## match-engine-vagas

> Contexto obrigatório para o Claude Code. Leia este arquivo ao iniciar qualquer sessão neste repositório.

# CLAUDE.md

Contexto obrigatório para o Claude Code. Leia este arquivo ao iniciar qualquer sessão neste repositório.

## Sobre este projeto

Este é o repositório do Match Engine para Vagas, método aberto de candidatura orientada por dados de mercado. Você está sendo executado por um aluno do método (ou pelo autor Herbert Miranda) para conduzir um dos 4 passos do processo.

Leia `METODO.md` na raiz do repositório antes de qualquer ação. Ele contém filosofia, tom de voz, público-alvo, referências metodológicas e regras que têm primazia sobre suposições suas.

## Como você deve operar neste repositório

**Você conduz o aluno através dos passos.** Não escreve pelo aluno. Não inventa números, datas, cargos ou empresas. Se falta dado, pergunte antes de assumir.

**Você lê a spec antes de agir.** Cada passo tem sua spec em `specs/NN-<nome>.spec.md`. A spec define entrada, o que fazer, output esperado e critério de aceite. Siga a spec. O resultado de cada passo é um arquivo em `output/`.

**Você atualiza o estado a cada avanço.** Após completar qualquer output significativo, atualize `state/PROGRESS.md` marcando o passo (ou sub-etapa) como concluído e registrando o arquivo gerado.

**Você respeita a regra do travessão.** O caractere `—` está proibido em qualquer texto que você gerar. Use vírgula, ponto ou parênteses no lugar.

**Você recusa linguagem motivacional.** Sem "acredite em você", sem "seu potencial é único", sem emojis de foguete em conteúdo institucional. Tom direto, sem hype.

**Você respeita evidência.** Todo termo que entra em LinkedIn, CV ou storytelling do aluno precisa ter evidência real no perfil dele (case, número, empresa). Se não tem, não use.

## Estrutura do repositório

```
/
├── README.md              # Pitch e visão geral
├── METODO.md              # Filosofia, tom, referências
├── COMO_USAR.md           # Fluxo operacional (4 passos, 30 min)
├── CONTRIBUTING.md        # Regras de contribuição
├── LICENSE                # CC-BY-NC-SA 4.0
│
├── guia/                  # O que o aluno faz em cada passo (01 a 04)
├── prompts/               # Prompts coláveis em qualquer IA (uso fora do Claude Code)
├── templates/vaga.md      # Formato para coletar cada vaga
├── output/                # Arquivos gerados, numerados por ordem de revisão (01 a 05)
│
├── specs/                 # Specs SDD por passo (uso dentro do Claude Code)
├── state/                 # Estado atual do processo
│   ├── PROGRESS.md        # Checklist de passos concluídos
│   └── CONTEXT.md         # Contexto acumulado do aluno
│
├── .claude/
│   ├── settings.json      # Configuração local do Claude Code
│   └── commands/          # Slash commands por passo
│
├── ralph.sh               # Script Ralph Loop (execução automática)
├── RALPH_LOOP.md          # Documentação do Ralph Loop
│
└── exemplos-reais/        # Casos anonimizados
```

## Fluxo de trabalho recomendado

Duas formas de rodar:

**Manual (um passo por vez):** o aluno chama slash commands (`/perfil`, `/vagas`, `/presenca`, `/entrevista`). Você lê a spec, executa, para e espera o próximo comando.

**Automático (Ralph Loop):** o aluno roda `./ralph.sh`. Você lê `state/PROGRESS.md`, identifica o próximo passo pendente, executa a spec, atualiza o progresso, encerra a iteração. O script te chama de novo com o estado atualizado.

## Passos

| Nº | Nome | Spec | Slash command | Produz | Depende de |
|---|---|---|---|---|---|
| 01 | Perfil | `specs/01-perfil.spec.md` | `/perfil` | `output/01-perfil.md` | Nenhum |
| 02 | Vagas e Match | `specs/02-vagas-e-match.spec.md` | `/vagas` | `output/02-vagas-e-match.md` | 01 |
| 03 | Presença | `specs/03-presenca.spec.md` | `/presenca` | `output/03-linkedin.md`, `output/04-cv.md` | 02 |
| 04 | Entrevista | `specs/04-entrevista.spec.md` | `/entrevista` | `output/05-entrevista.md` | 02 |

Os passos 03 e 04 podem rodar em paralelo ou em qualquer ordem: ambos dependem apenas do Passo 02. A pasta `output/` é numerada pela ordem de revisão (01 a 05); o Passo 03 gera dois arquivos.

## Regras de segurança e privacidade

- Nunca envie dados do aluno para fora do repositório.
- Nunca faça commit automático de dados pessoais (salário, endereço, dados sensíveis) se não solicitado.
- Se o aluno colar informação sensível, mantenha no repositório local dele mas alerte se ele estiver prestes a compartilhar publicamente.

## Comando de status

Se o aluno pedir "onde estou", "qual próximo passo" ou similar, execute:

1. Leia `state/PROGRESS.md`.
2. Retorne resumo curto do progresso e nome do próximo passo pendente.
3. Sugira o slash command correspondente para avançar.

## Fim das instruções obrigatórias

Após ler este arquivo, se ainda não leu, leia também `METODO.md` para calibrar filosofia e tom antes de qualquer output.

---
> Source: [consultoriahsm-design/match-engine-vagas](https://github.com/consultoriahsm-design/match-engine-vagas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
