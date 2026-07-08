## ai-services

> Convencoes para servicos de IA e integracoes com APIs externas


# Servicos de IA e Integracoes

## AI Gateway Pattern
- Toda comunicacao com modelos de IA passa pelo `AIGateway`
- O Gateway abstrai o modelo especifico e oferece interface uniforme
- Logs de uso sao registrados em `ai_usage_logs` (modelo, tokens, custo, usuario, marca)

## Modelos Suportados
- **OpenAI**: GPT-4o, GPT-4o-mini, DALL-E 3
- **Google**: Gemini 2.0 Flash, Gemini 2.0 Pro
- **Anthropic**: Claude 3.5 Sonnet, Claude 3.5 Haiku
- **Imagens**: DALL-E 3, Stability AI

## Contexto de Marca
- Toda chamada de IA deve incluir o contexto da marca ativa no system prompt
- Contexto inclui: tom de voz, segmento, publico-alvo, palavras-chave, cores
- O metodo `Brand::getAIContext()` retorna o contexto formatado

## Tratamento de Erros
- Retry automatico com backoff exponencial (3 tentativas)
- Fallback para modelo alternativo quando possivel
- Log detalhado de erros com request/response
- Rate limiting por modelo e por usuario

## Streaming (Chat)
- Usar Server-Sent Events (SSE) para streaming de respostas
- Endpoint dedicado para streaming: `/api/chat/stream`
- Frontend consome via `EventSource` ou `fetch` com `ReadableStream`

## Integracoes de Redes Sociais
- Cada plataforma tem seu proprio Service (`InstagramService`, `TikTokService`, etc.)
- Publicacao sempre via Queue Jobs (nunca sincrona)
- Tokens OAuth sao renovados automaticamente
- Erros de publicacao geram notificacao ao usuario

## Integracoes de Analytics
- Sync de dados e feito via scheduled jobs (a cada 6h ou diariamente)
- Dados sao normalizados para formato padrao antes de salvar
- Manter historico de snapshots para comparacao entre periodos

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
