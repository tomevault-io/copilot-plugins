## stalkeai

> - Sempre responda em **português brasileiro**.

# Instruções para o Claude

## Idioma
- Sempre responda em **português brasileiro**.

## Sobre o Projeto
**Stalkea** é uma aplicação web que simula uma ferramenta de análise de contas do Instagram e um clone da interface. É um projeto client-side only (sem backend real), com dados mockados e animações interativas.

### Fluxo Principal
1. Landing page com animação de digitação e contador de estatísticas
2. Input de username com modal de confirmação de perfil
3. Redirecionamento para Feed, Direct e páginas de Chat (Chat1-Chat5)

## Tech Stack
- **Framework:** React 19 (Create React App)
- **Roteamento:** React Router DOM v7
- **Estilização:** CSS Modules
- **Ícones:** FontAwesome 7
- **Deploy:** Vercel
- **Node:** 20.x

## APIs Externas
- `ipapi.co/json/` — geolocalização por IP
- `wtfismyip.com/json` — geolocalização alternativa
- `api.geonames.org` — cidades próximas (conta demo)
- `pravatar.cc` — geração de avatares

## Estrutura do Projeto
```
src/
├── pages/              # Páginas (Home, Feed, Direct, Chat1-5)
├── components/
│   ├── HomeComponents/     # HeroSection, MatrixCanvas, InstagramLogin, ConfirmModal
│   ├── ChatComponents/     # ChatBody, ChatHeader, ChatInput, ChatMessage*
│   ├── DirectComponents/   # DirectChatItem, DirectHeader, DirectSearch, DirectStories
│   └── FeedComponents/     # FeedHeader, FeedPost, StoriesBar, BottomNav
├── hooks/              # useGeolocation
├── utils/              # maskUsername
├── assets/             # Imagens (chat/, feed/, direct/)
├── App.js              # Roteamento principal
└── index.js            # Entry point
```

## Padrões do Código
- Componentes funcionais com Hooks (useState, useEffect, useRef)
- CSS Modules para escopo de estilos (`*.module.css`)
- Custom Hooks para lógica reutilizável
- Fetch API nativa (sem axios)
- Estado local com useState (sem Redux/Context)
- LocalStorage para persistência de estatísticas
- Props drilling para comunicação entre componentes

## Deploy
- Plataforma: **Vercel**
- Build: `CI=false NODE_OPTIONS=--max-old-space-size=4096 react-scripts build`
- Output: `/build`
- SPA routing configurado no `vercel.json`

## Comandos
```bash
npm start      # Dev server (porta 3000)
npm run build  # Build de produção
npm test       # Testes
```

---
> Source: [eduardokenzinho/stalkeai](https://github.com/eduardokenzinho/stalkeai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
