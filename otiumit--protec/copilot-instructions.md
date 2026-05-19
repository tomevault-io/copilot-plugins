## readme

> Lista consolidada de regras para o Cursor:

# Regras do Projeto - SaaS Boilerplate

Lista consolidada de regras para o Cursor:

1. **arquitetura-core**: Estrutura monorepo e modularidade (Always Apply).
2. **multitenancy**: Isolamento total de dados entre empresas (Always Apply).
3. **seguranca**: Autenticação, senhas e proteção de dados (Always Apply).
4. **backend-api**: Padrões Hono/Node.js, estrutura modular e Zod.
5. **frontend-react**: Padrões React, Tailwind e UI dinâmica.
6. **billing-modular**: Controle de planos, usuários e módulos.
7. **module-documentation**: Documentação obrigatória de módulos (README.md).
8. **database**: Configuração PostgreSQL (local ou Supabase com conexão direta).

## Observações Importantes

- **Remova regras antigas** (typescript, api-design, codigo-padroes) pois foram consolidadas nestas novas versões.
- **Sempre aplique** as regras de Multitenancy e Segurança (Always Apply).
- **Valide** todas as entradas do usuário.
- **Documente** decisões arquiteturais importantes.
- **Teste** isolamento de tenants em desenvolvimento.

---
> Source: [OtiumIT/protec](https://github.com/OtiumIT/protec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
