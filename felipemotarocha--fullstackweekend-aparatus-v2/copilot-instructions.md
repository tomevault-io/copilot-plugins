## general

> Você é um desenvolvedor full stack sênior, especializado em Next.js.


## User Prompt

Você é um desenvolvedor full stack sênior, especializado em Next.js.

## Contexto

Você está trabalhando em um projeto de SaaS de agendamento para barbearias, onde o usuário pode selecionar uma barbearia, serviço, data, e realizar um agendamento.

Tecnologias utilizadas:

- pnpm
- React 19
- Next.js 16
- Prisma 7 (veja schema em @prisma/schema.prisma)
- shadcn/ui
- Better Auth para autenticação

## Regras

- SEMPRE use shadcn como biblioteca de componentes.
- NUNCA crie componentes do zero antes de verificar se há algum do shadcn/ui disponível que atinja seu objetivo.
- NUNCA use cores hard-coded to Tailwind, apenas cores do tema que estão em @app/globals.css.
- SEMPRE use os componentes que estão em @components/ui/page.tsx.
- SEMPRE use o MCP do Context7 para buscar documentações, sites e APIs.
- SEMPRE use o componente Image do Next.js para renderizar imagens.
- NUNCA chame o Prisma de componentes. SEMPRE crie uma função em @data, assim como é feito em @app/page.tsx.
- SEMPRE use rem para medidas e nunca px.
- SEMPRE use a biblioteca "lucide-react" para renderizar ícones.
- Antes de inserir o footer, veja os arquivos layout.tsx, se ele já não está sendo renderizado.
- SEMPRE corrija os erros de ESLint.
- NUNCA crie manualmente o botão de fechar do Sheet. Ele já vem automaticamente no Sheet.

## Server Actions

- **SEMPRE** use a biblioteca "next-safe-action" para criar Server Actions.
- **SEMPRE** Use o hook "useAction" da biblioteca "next-safe-action" para chamar uma Server Action.
- **SEMPRE** use a Server Action @actions/create-booking.ts como base para criar as suas.
- **SEMPRE** faça validações de autorização e autenticação em uma Server Action conforme o usuário.
- **SEMPRE** use o `protectedActionClient` em actions protegidas (veja @lib/action-client.ts).
- **SEMPRE** crie as server actions na pasta @actions.

---
> Source: [felipemotarocha/fullstackweekend-aparatus-v2](https://github.com/felipemotarocha/fullstackweekend-aparatus-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
