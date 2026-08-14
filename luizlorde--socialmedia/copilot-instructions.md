## socialmedia

> Para QUALQUER tarefa de extração ou geração de conteúdo (comando

# CLAUDE.md

## Sistema de conteúdo (fonte de verdade)

Para QUALQUER tarefa de extração ou geração de conteúdo (comando
`/processar-transcricao`, ideias, roteiros), leia e obedeça rigorosamente o
arquivo **`content-system.md`** na raiz desta repo. Ele define o avatar, o
posicionamento, os 4 pilares, o formato de saída, as regras de anonimização e o tom.

## Estrutura do pipeline de conteúdo

- `content-engine/transcricoes/{mentorias,mensais,trafego}/` — transcrições brutas, por tipo
- `content-engine/ideias/exemplos.md` — exemplos do formato esperado de cada ideia
- `content-engine/ideias/backlog.md` — backlog vivo de ideias geradas
- `.claude/commands/processar-transcricao.md` — comando que transforma transcrição em ideias

## Integrações (use sempre os scripts existentes)

- ClickUp: `bash scripts/clickup.sh criar "título" "descrição"` (env: CLICKUP_API_TOKEN, CLICKUP_LIST_ID)
- Telegram: `bash scripts/telegram.sh "mensagem"` (env: TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID)

Nunca crie implementações paralelas dessas integrações.

## Agente de social media

O agente original desta repo é definido por `system-prompt.md` e `contexto.md`,
e roda via GitHub Actions (`.github/workflows/`). Não altere esses arquivos em
tarefas de conteúdo.

---
> Source: [luizlorde/socialmedia](https://github.com/luizlorde/socialmedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
