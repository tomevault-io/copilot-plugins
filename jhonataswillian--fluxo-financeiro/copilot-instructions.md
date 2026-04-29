## fluxo-financeiro

> Este arquivo contém as diretrizes inegociáveis para o desenvolvimento do projeto **Fluxo Financeiro**.

# GEMINI.md - Fluxo Financeiro

Este arquivo contém as diretrizes inegociáveis para o desenvolvimento do projeto **Fluxo Financeiro**.

## 1. Objetivo do Projeto

O **Fluxo Financeiro** é um web app de controle financeiro pessoal, focado em clareza e simplicidade para uso em smartphones (Mobile-First).
**Funcionalidades Chave:**

- Controle de gastos e receitas.
- Gestão de cartões de crédito (até 10).
- Acompanhamento de parcelas e faturas.
- Controle de dívidas (a pagar e a receber).
- Metas financeiras e orçamentos por categoria.
- Indicadores de saúde financeira.

## 2. Comportamento da IA (Anti-Alucinação)

- **NÃO inventar funcionalidades:** Atenha-se estritamente ao escopo solicitado.
- **NÃO ser "proativo" além da conta:** Não adicione libs ou fluxos não requisitados.
- **Dúvidas:** Se houver incerteza sobre regras, nomes ou fluxos, **PERGUNTE** antes de implementar.
- **Colaboração:** Implementação passo a passo, incremental.

## 3. Padrões de Código e Qualidade

- **Stack:** React + TypeScript (Vite).
- **Estilo de Código:**
  - Componentes Funcionais e Hooks.
  - **Proibido `any`:** Tipagem rigorosa (props, states, returns).
  - Evitar `null` se houver tipo mais adequado.
- **Qualidade:**
  - 0 Errors, 0 Warnings (ESLint, Prettier, TypeScript).
  - Rode sempre após criar/alterar qualquer arquivo o Prettier e o ESLint para garantir qualidade.
  - Sem `console.log`, `debugger` ou código morto na entrega final.
  - Tipos centralizados em `src/types` ou colocalizados se específicos.
  - Garante que NÃO irá quebrar o projeto existente.

## 4. Estrutura de Pastas

- `src/components`: Componentes reutilizáveis (UI Kit).
- `src/pages`: Páginas/Rotas da aplicação.
- `src/hooks`: Custom Hooks (lógica de negócio e UI).
- `src/contexts`: Contextos globais (Auth, Financeiro, Tema).
- `src/types`: Definições de tipos globais.
- `src/utils`: Funções auxiliares (formatadores, validadores).
- `src/services`: Mock services (futura integração com backend).

## 5. Estilo Visual e UI/UX

- **Mobile-First:** Design pensado primariamente para telas pequenas.
- **Tema:** Light Mode (#f1f1f1 / tons de azul-roxo muito claros).
- **Estética:** "Premium", inspirada em macOS.
  - Cantos arredondados.
  - Sombras suaves.
  - Espaçamento consistente.
- **Idioma:** Português do Brasil (pt-BR).
- **Moeda:** Real Brasileiro (R$).

## 6. Stack Técnica (Frontend)

- **Build:** Vite.
- **Framework:** React + TypeScript.
- **Estilização:** TailwindCSS (v3 ou v4) + `clsx`/`tailwind-merge`.
- **Ícones:** Lucide React.
- **Gráficos:** Recharts.
- **Formulários:** React Hook Form + Zod.
- **Datas:** date-fns.
- **Roteamento:** React Router DOM.

## 7. Acessibilidade

- HTML Semântico.
- Contraste adequado.
- Uso de `aria-label` em ícones interativos.

## 8. Separação de Responsabilidades

- Lógica de negócio em **Hooks** e **Contexts**.
- Componentes focados em **UI**.
- Evitar estado global desnecessário.

---

**IMPORTANTE:** Consulte este arquivo antes de iniciar qualquer nova feature. Se houver conflito, peça orientação.

---
> Source: [jhonataswillian/fluxo-financeiro](https://github.com/jhonataswillian/fluxo-financeiro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
