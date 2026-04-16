## dbgate-ai

> Plugin AI Chat intégré dans DbGate via injection. Un script lanceur extrait l'app.asar de DbGate, patche son main process pour ajouter un `WebContentsView`, et lance DbGate modifié avec un panneau IA latéral.

# CLAUDE.md — DbGate AI Chat Plugin

## Projet

Plugin AI Chat intégré dans DbGate via injection. Un script lanceur extrait l'app.asar de DbGate, patche son main process pour ajouter un `WebContentsView`, et lance DbGate modifié avec un panneau IA latéral.

Stack : **Vue 3** (Composition API, `<script setup>`) + **Pinia** + **Tailwind CSS v4** + **Vite**.

## Commandes

```bash
pnpm build    # Build de production (renderer → dist/)
pnpm dbgate   # Build + lance DbGate avec le plugin injecté
```

## Architecture

```
launcher/               # Injection dans DbGate (CommonJS, exécuté dans le main process de DbGate)
  launch-dbgate.js      # Extrait app.asar, patche electron.js, lance DbGate modifié
  ai-panel.js           # WebContentsView : toggle, resize, splitter, CSS overrides
  ai-chat-ipc.js        # IPC handlers préfixés aichat: (connexions, chat, export, etc.)
  preload-ai.js         # Preload pour le WebContentsView (contextBridge → window.api)
  inject-toggle.js      # JS injecté dans le renderer DbGate (bouton icon bar + splitter)
src/                    # Backend (CommonJS) — NE PAS MODIFIER sauf demande explicite
  config.js             # Gestion config JSON
  tabs-store.js         # Persistance des tabs
  ai/                   # Logique OpenAI (agentic loop, streaming)
  db/                   # Drivers SQL (pg, mysql2, better-sqlite3)
  dbgate/               # Import connexions DbGate, lecture thème
renderer/               # Frontend Vue 3 (ESM)
  index.html
  main.js               # createApp + Pinia
  App.vue               # Layout principal + IPC listeners
  assets/main.css       # Tailwind @theme + @layer base/components + animations
  utils/markdown.js     # Renderer markdown custom (ESM)
  stores/               # Pinia stores
    connections.js       # Connexions, statuses
    schema.js            # Active DB, schema tree
    chat.js              # Tabs, messages, streaming, models, writeMode
    settings.js          # Config, API key
  components/
    chat/                # TabBar, TabItem, TabContextMenu, ChatPanel, ChatMessages,
                         # MessageBubble, AgentResponse, AgentStep, ChatInput
    modals/              # SettingsModal, WriteConfirmModal
    shared/              # MiniTable, MarkdownContent
data/                   # Runtime data (config.json, tabs.json)
dist/                   # Build output (Vite)
```

## IPC (window.api)

API exposée via `launcher/preload-ai.js` (contextBridge). Tous les channels sont préfixés `aichat:`.

**Invocations :** `listConnections`, `connect`, `disconnect`, `listDatabases`, `getSchema`, `executeQuery`, `sendMessage`, `listModels`, `getConfig`, `saveConfig`, `testApiKey`, `loadTabs`, `saveTabs`, `exportCSV`, `exportJSON`, `getDbgateTheme`, `respondWriteConfirm`

**Events (listeners) :** `onChatToken`, `onAgentStep`, `onAgentStatus`, `onConnectionStatuses`, `onConnectionStatusSingle`, `onWriteConfirm`

## Injection dans DbGate

Le launcher patche le `src/electron.js` de DbGate avec deux insertions :
1. `const aiPanel = require('PLUGIN_DIR/launcher/ai-panel');` (après les requires)
2. `aiPanel.setup(mainWindow, ipcMain, 'PLUGIN_DIR');` (après `loadMainWindow()`)

`ai-panel.js` crée un `WebContentsView` chargé depuis `dist/index.html`, gère le toggle (bouton robot dans l'icon bar), le resize (splitter draggable), et les CSS overrides (`--dim-content-right`).

## Règles CSS — IMPORTANT

### Toujours utiliser Tailwind CSS, jamais de CSS custom

- **Utiliser les classes utilitaires Tailwind** dans les templates Vue pour TOUT le styling
- **Ne JAMAIS écrire de CSS custom** sauf cas exceptionnels (animations `@keyframes`, styles pour du contenu `v-html`)
- Le fichier `main.css` contient uniquement :
  - `@import "tailwindcss"` et le `@theme` (design tokens)
  - `@layer base` pour body et scrollbar
  - `@layer components` pour les styles markdown/code-blocks (nécessaires car `v-html`)
  - Les `@keyframes` pour les animations custom

### Tailwind v4 — Pièges à éviter

- **Ne JAMAIS écrire de CSS en dehors d'un `@layer`** (sauf `@keyframes`). En Tailwind v4, les styles non-layered écrasent toutes les classes Tailwind.
- Les couleurs custom sont dans `@theme` : `bg-bg-primary`, `text-text-primary`, `border-border`, `text-accent`, etc.
- Pour des valeurs pixel exactes : `py-[5px]`, `pl-[19px]`, `text-[11px]`
- Notre animation custom s'appelle `status-pulse` (pas `pulse` qui est natif Tailwind).
- Utiliser `@layer components` pour cibler du contenu `v-html`.

## Patterns Vue

### Composition API uniquement

Toujours `<script setup>`. Pas d'Options API.

### Markdown : `v-html` + event delegation

```vue
<div @click="handleClick" v-html="renderedHtml" />
```
`handleClick` détecte `.btn-copy` via `e.target.closest()`.

### IPC listeners : enregistrés une fois dans `App.vue` onMounted

```js
window.api.onChatToken(token => chatStore.appendToken(token))
```

### Streaming : RAF-throttled

Les tokens s'accumulent dans un buffer non-réactif, flush via `requestAnimationFrame` (~60fps). Voir `chat.js` store (`appendToken`, `streamRenderCallback`).

### Tab persistence

`chatStore.persistTabs()` est debounced (500ms), sérialise en JSON plain (pas de Proxy Vue), et sauvegarde via `window.api.saveTabs()`.

## Conventions

- **Langue du code :** anglais
- **Composants :** PascalCase, un composant par fichier `.vue`
- **Stores :** un fichier par domaine dans `renderer/stores/`
- **Imports :** utiliser `@/` comme alias pour `renderer/`
- **Pas de sur-engineering :** pas d'abstraction inutile, pas de code mort

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AirOne-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
