## ucp-supabase

> Boas práticas para Supabase (RLS, chaves, migrations e supabase-js)


# Supabase UCP

Ao integrar com Supabase (Auth, Postgres, Storage) e usar `@supabase/supabase-js`, aplique:

- Chaves/segredos: nunca exponha `SERVICE_ROLE_KEY` no frontend; use no backend apenas. Nunca logue chaves/tokens.
- RLS: ative e modele `Row Level Security` para isolar dados por tenant/usuário. No backend, aplique as checagens de permissão.
- Migrations: mudanças de schema via migrations/versionamento; evite `ALTER` ad-hoc sem controle.
- Queries: selecione colunas necessárias, pagine resultados e evite `select *`.
- Consistência: trate erros de query explicitamente e retorne mensagens seguras.
- Resiliência: em integrações críticas, implemente retry apenas para erros transitórios; preserve timeouts.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
