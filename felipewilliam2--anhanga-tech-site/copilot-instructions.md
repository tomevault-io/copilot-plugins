## anhanga-tech-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este repositório

Site institucional (single-page) da anhangá.tech, uma agência de automação de negócios com IA voltada para pequenos negócios. React 19 + Vite + TypeScript + Tailwind v4. Sem backend — é um site estático de marketing/apresentação.

O plano de produto e negócio completo (modelo de negócio, produtos, precificação, posicionamento, decisões de arquitetura de conteúdo) vive em `PRODUCT.md` — consulte-o antes de propor mudanças de copy, estrutura de páginas ou novos produtos/seções, pois ele é a fonte de verdade dessas decisões. O backlog de execução vive nas issues do GitHub, não em arquivos locais.

## Comandos

Gerenciador de pacotes: **pnpm** (via `corepack enable`). Node 24.

- `pnpm install` — instalar dependências
- `pnpm dev` — servidor de desenvolvimento (porta 3000, `host: 0.0.0.0`)
- `pnpm typecheck` — `tsc --noEmit`
- `pnpm build` — build de produção (`vite build`)
- `pnpm preview` — preview do build

Não há suíte de testes nem linter configurado. O CI (`.github/workflows/ci.yml`) roda apenas `pnpm typecheck` e `pnpm build` em push/PR para `main` — trate esses dois comandos como o gate mínimo de validação antes de considerar uma mudança pronta.

## Arquitetura

- **`App.tsx`** — single-file: contém a página inteira (~760 linhas), incluindo todas as seções (hero, serviços, planos, metodologia, formulário de contato, newsletter etc.) como um único componente `App`. Os dados de conteúdo (cards de serviço, planos, passos da metodologia) estão declarados como arrays no topo do arquivo, antes do componente. Ao editar uma seção, é mais rápido localizá-la por texto/copy do que por estrutura de arquivos.
- **`components/`** — poucos componentes compartilhados extraídos do `App.tsx` (`Sidebar`, `GridCard`, `Marquee`). Novos componentes reutilizáveis devem seguir esse padrão: função nomeada exportada, tipada com `React.FC` e uma interface `*Props` local.
- **`index.tsx`** — bootstrap padrão do React 19 (`createRoot`), monta `<App />` em `#root`, importa `index.css`.
- **`index.css`** — tema Tailwind v4 via `@theme` (cores de marca `anhanga-*`, fontes, keyframes de animação). Utilitários customizados (`.grid-card`, `.text-outline`, `.no-scrollbar`) ficam em `@layer components`/CSS puro no mesmo arquivo — não há outro arquivo de estilos.
- **`types.ts`** — tipos compartilhados mínimos (`ServiceCardProps`, `NavItem`).
- Alias de import `@/*` aponta para a raiz do projeto (configurado em `tsconfig.json` e `vite.config.ts`).
- Tailwind é integrado via plugin do Vite (`@tailwindcss/vite`), sem `tailwind.config.js` — a configuração de tema vive inteiramente em `index.css`.

## Convenções observadas

- Paleta de marca definida em `index.css` como variáveis `--color-anhanga-*` (green, lime, stone, dark, accent) — usar essas classes utilitárias (`bg-anhanga-dark`, `text-anhanga-lime` etc.) em vez de cores arbitrárias do Tailwind ao trabalhar em UI da marca.
- Acessibilidade é levada a sério no CSS base: `prefers-reduced-motion` desativa animações, e há estilos de foco reforçados (`focus-visible`) — preservar esse comportamento em novos componentes interativos.

---
> Source: [felipewilliam2/anhanga-tech-site](https://github.com/felipewilliam2/anhanga-tech-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
