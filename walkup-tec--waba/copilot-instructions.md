## backend-saas-api-standards

> Padrões de APIs backend (Clean Architecture + segurança + tenant_id quando aplicável)


# Backend SaaS API Standards

Ao trabalhar com rotas/handlers HTTP e lógica de backend em TypeScript (Express ou similar), siga Clean Architecture:

- `controller`: valida/normaliza input e monta resposta; chama `service`.
- `service`: regra de negócio, autorizações, transações e orquestração.
- `repository`: acesso a dados (queries/ORM) sem lógica de negócio.

Segurança e robustez:

- Trate erros corretamente (400/401/403/404/409/500) e nunca exponha detalhes sensíveis.
- Valide todas as entradas antes de chamar `service`/`repository`.

Multi-tenant (se existir `tenant_id` no domínio):

- Propague `tenant_id` do controller para service e repository.
- Garanta isolamento: todas leituras/escritas devem filtrar por `tenant_id` ou validar pertencimento antes de mutar.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
