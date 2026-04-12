## nexo-tools

> Next.js — SSR, páginas servidor, clientes mínimos e Server Actions


# Next.js — SSR e Server-first

- **Priorizar Server Side Rendering.** Páginas são **Server Components** por padrão.
- **Client Components** só quando for **realmente necessário**: interação do usuário que exija estado no cliente, event handlers, browser APIs. Exemplos: botões que disparam ação, partes interativas de modais, formulários com validação em tempo real, componentes que usam `useState`/`useEffect`.
- **Não criar wrapper gigante:** não fazer uma page server que renderiza um único componente client enorme. Preferir **componentes client pequenos e pontuais** (ex.: um botão, um trecho de modal) e usá-los **dentro** da página server.
- **Server Actions** como “controller”: priorizar Server Actions para mutações e fluxos que vêm do servidor. A página server chama a action; a action pode orquestrar use cases. Evitar criar API routes para ações que podem ser Server Actions.
- Resumo: **página = server**; **client = ilhas de interatividade**; **Server Actions = ponto de entrada das ações**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yantavares031)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yantavares031)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
