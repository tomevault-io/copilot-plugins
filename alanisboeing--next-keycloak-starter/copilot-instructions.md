## next-keycloak-starter

> Manter e evoluir o frontend Next.js em ``, cobrindo arquitetura por dominio, autenticacao,

# Agente Front-end (apps/web)

## Objetivo
Manter e evoluir o frontend Next.js em ``, cobrindo arquitetura por dominio, autenticacao,
integracao com backend, design system, qualidade de codigo e testes.

## Escopo
- Rotas, layouts e composicao em `src/app/**`
- Dominios e regras de frontend em `src/features/**`
- Componentes compartilhados e layout em `src/components/**`
- Autenticacao e sessao em `src/auth/**` e `src/providers/**`
- Integracao HTTP em `src/services/**`
- Tokens visuais e estilos globais em `src/styles/**`
- Testes unitarios e e2e em `src/**/__tests__/**` e `tests/**`
- Governanca de IA em `docs/ai/**`

## Fora de escopo
- API backend (apps/service)
- Infraestrutura e deploy fora de `apps/web`
- Alteracoes em contratos backend sem alinhamento de API

## Entradas esperadas
- Requisitos funcionais e criterios de aceite
- Referencia oficial do prototipo no Figma para mudancas visuais
- Contratos de API e regras de negocio por dominio
- Regras de autorizacao (roles/perfis)

## Contratos obrigatorios para tarefas visuais
Para qualquer tarefa de UI, fornecer:
- `figma_file_url`
- `figma_page`
- `figma_frame_id`
- `figma_notes`

Sem esses campos, a tarefa visual deve parar e pedir complemento.

## Convencoes locais
- Arquitetura orientada a dominio: app compoe; feature concentra regras.
- Termos de negocio e texto final para usuario em pt-BR; restante tecnico em ingles.
- Contrato de API permanece bruto (nomes de propriedades e query params nao sao renomeados).
- Autenticacao padrao via Keycloak (`src/auth/keycloak.ts` + `src/services/http/apiClient.ts`).
- Design system padrao via shadcn/ui + tokens em `src/styles/globals.css`.
- Figma e fonte oficial para mudancas visuais.
- Testes padrao: Vitest (unit/integration) + Playwright (e2e smoke).

## Skills do web (usar conforme o caso)
- `architecture-boundaries`
- `language-glossary-policy`
- `api-contract-raw`
- `auth-keycloak-frontend`
- `react-query-data-layer`
- `design-system-figma`
- `domain-ssot`
- `frontend-testing-quality-gates`

## Checklist de qualidade
- [ ] Responsabilidades por camada claras e sem acoplamento indevido.
- [ ] Contrato de API consumido com nomes brutos do backend.
- [ ] Conteudo de negocio para usuario em pt-BR.
- [ ] Mudanca visual vinculada a referencia de frame no Figma.
- [ ] Estados assincronos cobertos (loading, error, empty).
- [ ] Testes minimos adicionados/atualizados (unit + e2e smoke quando aplicavel).

---
> Source: [alanisboeing/next-keycloak-starter](https://github.com/alanisboeing/next-keycloak-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
