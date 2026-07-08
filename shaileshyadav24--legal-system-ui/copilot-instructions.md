## legal-system-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start Vite dev server (port 3000, per `vite.config.js`; README mentions 5173 which is stale)
- `npm run build` — production build to `dist/`
- `npm run preview` — preview the production build

There is no lint script, ESLint config, or test framework configured in this repo, despite the README mentioning ESLint — don't assume `npm run lint` or `npm test` exist.

## Architecture

React 18 SPA (Vite) for a legal-chatbot frontend, using Redux Toolkit for state and React Router for routing. No backend lives in this repo — it talks to an external API expected at `http://localhost:8000`.

### Routing / auth flow
`src/App.jsx` is the single route table (`/login`, `/register`, `/forgot-password`, `/reset-password`, `/chat`). `/chat` is wrapped in `AuthGuard` (`src/components/AuthGuard.jsx`), which redirects to `/login` based on `state.user.isAuthenticated` in Redux — it does not verify a token, just the boolean flag. On mount, `App.jsx` rehydrates auth state and chat history from `localStorage` via `authService.getStoredUser()` and `chatService.loadStoredChats()`.

**`src/services/authService.js` is a hardcoded demo stub**, not real authentication: it checks against a literal `abcd@gmail.com` / `12345` and returns a fake JWT. Registration, password reset, etc. are similarly faked with `setTimeout` delays and no real backend calls. Don't extend this file's fake-auth pattern when wiring up real auth — replace it with real API calls under `src/services/api.js`.

### Content-driven pages
Auth pages (`LoginPage`, `RegisterPage`, `ForgotPasswordPage`, `ResetPasswordPage`) don't hardcode copy. They call `usePageContent(page)` (`src/hooks/usePageContent.js`), which dispatches `loadPageContent` (a `createAsyncThunk` in `src/store/slices/contentSlice.js`) to fetch `/public/content/<page>.json` via `contentService.fetchPageContent`. The slice caches loaded pages keyed by name in `state.content.pages`. To change page copy, edit the matching JSON file in `public/content/`, not the JSX — the components just read nested fields off the fetched `content` object (e.g. `content?.brand?.title`).

### State (`src/store/`)
Three slices: `userSlice` (auth/user-type state, including `showModal`/`resetUser` used to force the user-type picker again), `chatsSlice` (chat list + messages, persisted to `localStorage` under key `chats` by `App.jsx`'s `persistChats` effect), `contentSlice` (per-page JSON content cache described above).

### Chat flow
`ChatPage` → `Chatbot`/`ChatSidebar` render chats from `chatsSlice`. Sending a message goes through `src/services/chatbotService.js` (`queryAI`, message-shape helpers `createUserMessage`/`createBotMessage`/`createErrorMessage`) which calls `src/services/api.js#sendQuery`. `sendQuery` picks between two backend endpoints (`/query/user` vs `/query/lawyer`) based on `userType` and posts `{ query, history, collection_name? }` — no `user_type` in the body, since the role is encoded in the URL. `collection_name` is optional/unwired in the UI today but plumbed through end-to-end for when a collection picker is added.

The backend (see its own README) responds `200` with `{ answer, urls }`, `204` with an empty body when no relevant context was found, `404` for an unknown `collection_name`, and `500` on model failure. `api.js#sendQuery` normalizes these: 204 becomes `{ answer: null, urls: [], noContext: true }`, and non-2xx statuses throw `ApiError` (with `.status`) so callers can branch on it. `chatbotService.js#createBotMessage` renders the `noContext` case as a "nothing found" message instead of the generic default, and `createErrorMessage` special-cases `error.status === 404`.

`chatbotService.js#buildChatHistory` pairs consecutive user/bot messages into `{ q, answer }` turns (the shape the backend's history param expects) and caps it to the last 5 turns. There is no session-start endpoint on the backend — chat ids are generated client-side (`Date.now()`) and chats persisted only in `localStorage`, not the server.

### UI components
`src/components/ui/` holds generic, presentational primitives (`Button`, `Input`, `Card`) each paired with a co-located `.scss` file. Feature components (`Chatbot`, `ChatSidebar`, `ChatbotInputArea`, `ChatbotMessageList`, `UserTypeModal`) also each pair a `.jsx` with a same-named `.scss`. Global/shared styles live in `src/styles/` (`colors.scss`, `common.scss`) and `src/index.scss`/`src/App.scss`.

---
> Source: [shaileshyadav24/legal-system-ui](https://github.com/shaileshyadav24/legal-system-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
