## ucp-docker

> Boas práticas para Docker/Docker Compose no projeto


# Docker UCP

Ao trabalhar com `Dockerfile` e `docker-compose.yml` (ou quando o pedido mencionar Docker/containers), priorize:

- Segurança de build: use imagens base oficiais e mantenha tags específicas (evite `latest`).
- Privilégios: rode como usuário não-root quando possível.
- Multi-stage build: minimize tamanho final e tempo de build.
- Segredos: nunca embuta tokens/chaves no Dockerfile; use `env var`/`secrets` do ambiente.
- Configuração: mantenha `PORT`, `TZ` e variáveis sensíveis fora do repositório.
- Persistência: defina volumes para dados necessários (ex.: Postgres, Supabase, n8n).
- Saúde e resiliência: inclua `healthcheck` e estratégias de restart adequadas.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
