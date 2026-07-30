## efka

> React + Vite frontend, Admin UI (3000) + User UI (3001).

# Frontend

React + Vite frontend, Admin UI (3000) + User UI (3001).

## Components

```
src/components/
├── ChatView.jsx       # Admin interface
└── UserChatView.jsx   # User interface
```

## Manual Start

```bash
npm run dev                                        # Admin :3000 (strict port)
npm run dev -- --port 3001 --config vite.user.config.js     # User :3001
```

Note: Vite is configured with `strictPort: true` to prevent silent port fallback. If `:3000`/`:3001` is occupied, free the port and retry.

## Dependencies

```bash
npm install
```

---
> Source: [Harryoung/efka](https://github.com/Harryoung/efka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
