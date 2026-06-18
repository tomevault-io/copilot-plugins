## the-maze-quest

> Contexto persistente para o Claude Code neste repositorio.

# CLAUDE.md

Contexto persistente para o Claude Code neste repositorio.

## Proposito
The Maze Quest — aplicacao narrativa Web3/crypto ambientada no universo da Paralax Corp (lore distopica corporativa, ano 2047, "Human Efficiency Protocol HEP-9"). Centrada no token $MAZE (Solana, supply 1B): 40% queimado quando o primeiro jogador completa a ultima Main Quest, 10% (100M) destinado a airdrop comunitario distribuido via leaderboard Zealy.

## Stack
- **Build/dev**: Vite 5 + `@vitejs/plugin-react` + `vite-plugin-node-polyfills` (Buffer/process para `@solana/web3.js` no browser).
- **Frontend**: React 19 + `react-router-dom` v7.
- **Wallet**: `@solana/wallet-adapter-react` / `@solana/wallet-adapter-wallets` / `@solana/web3.js`.
- **Backend leve**: Express (`server/index.js`) — proxia chamadas `/api/*` (Zealy leaderboard/quests/categories/user, Twitter OAuth 2.0 PKCE).
- **Dev concorrente**: `concurrently` roda Vite + Express juntos. Vite proxy `/api` -> `http://localhost:3001`.

## Estrutura
- `src/main.jsx` — bootstrap React + router.
- `src/LandingPage.jsx` — boot sequence ASCII estilo terminal Paralax Corp v4.7.2, exige codigo de acesso `themaze` para liberar a rota principal.
- `src/AboutPage.jsx` — docs estilo GitBook com sidebar, secoes: `overview`, `narrative`, `tokenomics`, `quests`, `airdrop`, `ritual`, `roadmap`, `community`.
- `src/App.jsx` — UI principal da quest: header com CA, barras de burn/airdrop, conexao de wallet Solana, badge X/Twitter, stats Zealy (XP, quests, rank), abas Quests / Leaderboard / Lore. Calculo de airdrop em duas tiers (Tier 1: 50M proporcional ao XP; Tier 2: 50M para Top 100, 5M fixos para o #1, restante distribuido progressivamente).
- `index.html` — title "The Maze Quest", theme-color `#040407`, favicon SVG.
- `server/` — endpoints Zealy + auth Twitter.

## Comandos (package.json)
- `npm run dev` — Vite + Express em paralelo (frontend + API).
- `npm run dev:frontend` / `npm run dev:server` — rodar isolado.
- `npm run build` — `vite build`.
- `npm run preview` — preview do build.
- `npm start` — apenas o servidor Express.

## Variaveis de ambiente (.env, ver .env.example)
- `ANTHROPIC_API_KEY` — verificacao IA de side quests (screenshot/tweet).
- `TWITTER_CLIENT_ID` (+ opcional `TWITTER_CLIENT_SECRET`, `TWITTER_REDIRECT_URI`) — OAuth 2.0 PKCE para "Connect X".
- `VITE_SOLANA_RPC` — RPC privado opcional (Helius/QuickNode); fallback mainnet publico.
- `VITE_ZEALY_COMMUNITY_SUBDOMAIN` — default `themazequest`.
- `VITE_TOKEN_CONTRACT` — endereco do token apos lancamento (exibe "To Be Announced" se ausente).
- `ZEALY_CATEGORIES` — JSON suportado para Vercel (ver historico de commits).
- `PORT` — porta do Express (default 3001).

## Deploy
Vercel — pasta `.vercel/` presente (`project.json` linkado). Variaveis sensiveis devem ser configuradas no painel da Vercel.

## Estado atual
- Branch: `master` (sincronizado com `origin`).
- Commit mais recente (`4b0e313`): adicao da Landing page com codigo de acesso + About page e integracao do react-router.
- Commits anteriores: suporte a `ZEALY_CATEGORIES` em JSON, remocao do `.env` do tracking, initial commit.

## Convencoes
- Estilos definidos inline via `<style>{STYLE}</style>` em cada pagina (CSS-in-JS string), com paleta amber/dark e fontes `Cinzel Decorative`, `EB Garamond`, `Share Tech Mono`.
- Tom narrativo: misterioso, distopico, "seek the light amidst the darkness".
- Nunca commitar `.env`. Usar `.env.example` como referencia.

---
> Source: [LeemuueL/the-maze-quest](https://github.com/LeemuueL/the-maze-quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
