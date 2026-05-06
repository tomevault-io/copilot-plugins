## ucp-docker-compose-yml

> docker-compose.yml - portas, volumes, saúde e rede segura


# Docker Compose UCP

Ao editar `docker-compose*.yml`:

- Não expor segredos em `environment`/`volumes` no repositório; use `env_file` e/ou `.env` fora do commit.
- Defina `healthcheck` quando houver dependências (DB, Supabase, serviços externos).
- Use `restart` coerente com criticidade e evite loops sem saída.
- Atribua `networks`/isolamento quando fizer sentido para reduzir superfície.
- Fixe versões de imagens e evite `latest`.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
