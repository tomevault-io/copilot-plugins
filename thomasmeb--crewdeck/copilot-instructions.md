## crewdeck

> > Dashboard visuel local-first pour piloter vos agents Claude.

# CrewDeck

> Dashboard visuel local-first pour piloter vos agents Claude.

## Architecture

- **Local-first** : tout tourne sur la machine de l'utilisateur
- **Auth** : reutilise l'authentification Claude Code locale (Pro/Max)
- **Tunnel** : Cloudflare Tunnel pour acces distant securise

## Stack

- Runtime : Node.js 22+, TypeScript 5.2+
- Backend : Hono (serveur HTTP leger)
- Frontend : Next.js 15 (export statique) + React 19 + Tailwind 4 + Framer Motion
- DB : better-sqlite3 + drizzle-orm (SQLite local)
- Agents : @anthropic-ai/claude-agent-sdk (V2 preview)
- Tunnel : cloudflared (npm)

## Structure

```
src/
├── cli.ts              # Point d'entree CLI (bin: "crewdeck")
├── server/             # Serveur HTTP Hono
│   ├── index.ts        # Setup serveur + routes
│   ├── routes/         # API REST + SSE
│   └── middleware/      # Auth tunnel, CORS
├── core/               # Logique metier
│   ├── orchestrator.ts # Gestion des agents Claude
│   ├── scanner.ts      # Detection des projets git
│   ├── tunnel.ts       # Gestion du tunnel Cloudflare
│   └── providers/      # Abstraction SDK agent
│       ├── types.ts    # Interface AgentProvider
│       └── claude-v2.ts # Adaptateur Claude Agent SDK V2
├── db/                 # SQLite + drizzle schema
│   ├── schema.ts       # Tables : projects, tasks, ideas, agents, messages, events
│   └── index.ts        # Connexion + migrations
└── web/                # Frontend Next.js
    ├── app/            # App Router pages
    ├── components/     # Composants React
    └── hooks/          # Custom hooks (useAgentEvents, etc.)
```

## Conventions

- Commits en anglais
- Code comments en anglais
- 1 agent actif par projet (MVP)
- SSE pour le streaming temps reel (pas WebSocket)
- Abstraction AgentProvider obligatoire (pas d'import direct du SDK)

## Commandes

```bash
npm run dev          # Dev mode (frontend + backend)
npm run build        # Build production
npm run lint         # ESLint + tsc --noEmit
npx crewdeck         # Lancer en local
npx crewdeck --tunnel # Lancer avec tunnel
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThomasMeb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ThomasMeb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
