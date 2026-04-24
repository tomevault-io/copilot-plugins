## dungeon-master

> Narrador de RPG web com IA (Ollama). Monorepo com backend Express e frontend React.

# RPG Narrator

Narrador de RPG web com IA (Ollama). Monorepo com backend Express e frontend React.

## Como rodar

```bash
# Iniciar tudo (backend + frontend)
start.bat

# Ou separadamente:
cd backend && npm start     # API na porta 3001
cd frontend && npm start    # React na porta 3000
```

## Estrutura do Projeto

```
RPG/
├── backend/                      # API Express + MongoDB
│   ├── server.js                 # Entry point
│   ├── config/
│   │   ├── db.js                 # Conexao MongoDB Atlas (Mongoose)
│   │   └── passport.js           # Google OAuth (condicional)
│   ├── middleware/
│   │   └── auth.js               # Auth middleware (dev mode bypass)
│   ├── models/
│   │   ├── User.js               # Usuario (googleId, username)
│   │   ├── World.js              # Mundo (slug, lore, races, classes)
│   │   ├── Character.js          # Personagem AD&D 2e completo
│   │   ├── GameSession.js        # Sessao multiplayer por turnos
│   │   └── Message.js            # Mensagens do chat
│   ├── routes/
│   │   ├── auth.js               # Login dev + Google OAuth
│   │   ├── worlds.js             # CRUD mundos
│   │   ├── characters.js         # CRUD personagens
│   │   ├── sessions.js           # Sessoes multiplayer
│   │   └── chat.js               # Narracao e criacao via Ollama
│   └── seed/worlds.js            # Seed: Dark Sun, Ravenloft
│
└── frontend/src/                 # React (CRA)
    ├── App.js                    # Rotas
    ├── themes.js                 # Temas dinamicos por mundo
    ├── api/client.js             # Axios instance
    ├── contexts/AuthContext.js   # Auth context (dev mode)
    ├── components/
    │   ├── Layout.js/css         # Navbar + dice bar + main
    │   ├── ProtectedRoute.js     # Guard de rota
    │   ├── CharacterSheet.js/css # Ficha AD&D 2e (7 abas)
    │   ├── ChatMessage.js/css    # Balao de mensagem
    │   ├── ChatInput.js/css      # Input do chat
    │   ├── DiceBar.js/css        # Barra de dados (esquerda)
    │   └── DiceOverlay.js/css    # Overlay 3D (@3d-dice/dice-box)
    └── pages/
        ├── LoginPage/            # Login por username
        ├── WorldSelectPage/      # Selecao de mundo
        ├── CharacterSelectPage/  # Lista personagens
        ├── CharacterCreationPage/# Chat LLM + ficha lado a lado
        ├── SessionLobbyPage/     # Lobby de sessoes
        └── NarrationPage/        # Chat de narracao por turnos

## Fluxo de Navegacao

/login → /worlds → /worlds/:slug/characters → criar ou selecionar
  criar:      /worlds/:slug/characters/new
  selecionar: /worlds/:slug/sessions → /sessions/:id
```

## Stack

- **Frontend**: React, React Router, Axios, @3d-dice/dice-box
- **Backend**: Express, Mongoose, Passport.js, express-session
- **Banco**: MongoDB Atlas
- **IA**: Ollama (local)
- **Temas**: CSS Custom Properties (Dark Sun = deserto, Ravenloft = gotico)

## Internacionalizacao (i18n)

- Idioma padrao: **Portugues (pt-BR)**
- Idiomas suportados: Portugues, Ingles (en), Alemao (de), Espanhol (es)
- Todos os textos da UI devem usar chaves de traducao, nunca strings hardcoded
- Arquivos de traducao em `frontend/src/i18n/locales/{pt,en,de,es}.json`

## Convencoes

- Pages ficam em subpastas proprias: `pages/NomePage/NomePage.js`
- Components reutilizaveis ficam em `components/` com CSS co-localizado
- Auth em dev mode usa username (sem Google OAuth obrigatorio)
- Backend roda na porta 3001, frontend na 3000
- Variaveis de ambiente no `backend/.env` (nao versionado)
- Textos da interface sempre via sistema de traducao (i18n)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noobDevPRB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
