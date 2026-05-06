## integration-apis-resilience

> Integrações com APIs externas resilientes (adapters, retry, logs, fallback)


# Integração de APIs externas (resiliência)

Ao implementar ou refatorar chamadas a APIs externas (HTTP via `fetch/axios`, SDKs de terceiros como Supabase, webhooks e gateways), aplique:

- Desacoplamento: crie uma camada `adapter/client/gateway` (ou service de integração) e mantenha o core dependente de contratos internos.
- Retry seguro: backoff exponencial + jitter; retentar apenas erros transitórios (timeouts, 502/503/504). Para writes, usar `idempotency key`.
- Timeouts e limites: defina connect/read timeouts e controle payload/stream.
- Logs detalhados (sem segredos): logue operação, provider, status code, duração e `correlationId/traceId` quando existir; nunca logue tokens, headers de autorização ou bodies com PII.
- Circuit breaker (quando necessário): para provedores instáveis/latência recorrente.
- Fallback: cache curto, fila para reprocessamento ou operação assíncrona quando o provedor estiver indisponível; documente limites do fallback.

Tratamento de erros:
- Prever indisponibilidade e responder com mensagens seguras ao cliente.
- Sempre tratar falhas externas explicitamente (rede, parsing, 4xx/5xx).

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
