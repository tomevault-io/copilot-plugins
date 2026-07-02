## my-chat-web

> - This project is a `Next.js 14` App Router application for three AI tools:

# Project Guide For Future Agents

## Overview

- This project is a `Next.js 14` App Router application for three AI tools:
  - chat: `/chat`
  - image generation: `/gen-image`
  - translation: `/translate`
- UI stack: `React 18`, `Tailwind CSS`, `shadcn/ui`, `Radix UI`, `framer-motion`.
- State management: `Zustand` with `persist`.
- Primary deployment target: `Cloudflare Pages` with `@cloudflare/next-on-pages`.
- Internal models are served through Next edge routes backed by the Cloudflare `AI` binding.
- External models are called directly from the browser via SDK/HTTP using user-provided API keys stored in local state/localStorage.

## Run And Deploy

- Install: `pnpm install`
- Local dev UI: `pnpm dev`
- Cloudflare Pages build: `pnpm pages:build`
- Local Pages preview: `pnpm preview`
- Deploy to Cloudflare Pages: `pnpm deploy`
- Deploy to Cloudflare Pages dev preview branch: `pnpm deploy:dev`

## Environment And Platform Notes

- Cloudflare config is in `wrangler.toml`.
- Pages dev deployments are branch-based preview deployments. This repo uses `pnpm deploy:dev` to deploy to the `dev` branch preview environment.
- Edge API routes rely on `getRequestContext().env`, so cloud deployment assumptions are built into `app/api/**/route.js`.
- `.env.example` only covers some browser-side vars. Real production secrets for Cloudflare/Gemini/private routes come from Cloudflare env/bindings, not from this file.
- README explicitly states the project is intended for Cloudflare deployment only.

## Directory Map

- `app/`: App Router pages, layout, and API routes.
- `components/`: UI components, grouped by domain (`chat`, `navbar`, `sidebar`, `common`, `ui`, `anim`).
- `hooks/`: page/model/chat hooks.
- `service/`: request layer and provider integrations.
- `store/`: global Zustand stores.
- `utils/`: shared enums, message builders, stream parsing, model metadata, storage helpers.
- `provider/`: theme provider.

## Main User Flows

### 1. Chat

- Page entry: `app/chat/page.js`
- Main container: `components/chat/index.jsx`
- User input + submission orchestration: `hooks/useChat.js`
- Internal chat API: `service/api.js -> /api/chat`
- External chat APIs:
  - `service/openai.js`
  - `service/deepseek.js`
  - `service/grok.js`
  - `service/gemini.js`
  - `service/ollama.js`

### 2. Image Generation

- Page entry: `app/gen-image/page.js`
- Shares the same chat UI shell, but `type` switches behavior in `useChat`.
- Internal image API: `service/api.js -> /api/gen-image`
- Optional prompt pre-translation is handled client-side in `genImageWithApi(..., preTrans)`.

### 3. Translation

- Page entry: `app/translate/page.js`
- Internal translation API: `service/api.js -> /api/trans`
- Translation target selection is stored in `useModelStore.currentModelInfo.transTarget`.

## Frontend Architecture

### Root Layout

- `app/layout.js` wires:
  - theme provider
  - sidebar provider
  - navbar
  - global toaster
  - analytics helpers
  - decorative cursor/loading components

### Shared Chat Shell

- `components/chat/index.jsx` is the shared screen for all three tools.
- It initializes models with `useInitModel()`.
- It reads conversation state from `useChat(type)`.
- It handles auto-scroll and regenerate-last-user-message behavior.

### Chat Component Roles

- `chat-topbar.jsx`: top actions/context for current tool.
- `chat-list.jsx`: message list container.
- `chat-message.jsx`: message rendering, markdown/image/pending state.
- `chat-bottombar.jsx`: input area and submit actions.
- `chat-scroll.jsx`: floating scroll-to-bottom helper.

## State Management

State is centralized in `store/index.js`.

### `useModelStore`

- Stores:
  - fetched internal model lists by route key
  - current selected model info by route key
  - third-party provider config, including API keys and Ollama host/model
- Persisted under `model-store`.
- Route keys are derived by `mapPathToKey()` in `utils/router.js`.

### `useChatStore`

- Stores three separate histories:
  - `messages`
  - `transMessages`
  - `imgMessages`
- Also stores `preTrans` for image prompt pre-translation.
- Persisted under `chat-store`.
- History caps:
  - chat: 500
  - translate: 500
  - image: 100

### `useChatStatusStore`

- Stores transient request loading state only.
- Not persisted.

### `useUserStore`

- Stores local nickname only.

## Request/Data Flow

### Internal Models

1. Page loads shared chat shell.
2. `useInitModel()` calls `getModels(path.replace('/', ''))`.
3. `service/api.js` requests `/api/models`.
4. `app/api/models/route.js` returns model metadata from `app/api/utils/models.js`.
5. User submits input in `useChat(type)`.
6. Based on tool type, request goes to:
   - `/api/chat`
   - `/api/gen-image`
   - `/api/trans`
7. Response is stored in Zustand and rendered by shared chat UI.

### External Models

1. `useChat(type)` chooses provider via `getChatApi(currentModel.type)`.
2. API keys/hosts are read from local persisted store via `utils/storage.js`.
3. Provider clients stream directly in the browser.
4. Stream chunks are normalized into the shared append-chunk flow.

## API Routes

### `/api/chat`

- File: `app/api/chat/route.js`
- Runtime: `edge`
- Uses Cloudflare `env.AI.run(...)`
- `POST` uses streaming output.
- Default system prompt is hardcoded: "You are a helpful assistant. 尽量使用中文回答"

### `/api/gen-image`

- File: `app/api/gen-image/route.js`
- Runtime: `edge`
- Uses Cloudflare image models.
- Some models return base64 strings and are normalized in-route.

### `/api/trans`

- File: `app/api/trans/route.js`
- Runtime: `edge`
- Uses Cloudflare translation model and returns translated text JSON.

### `/api/chat_gemini`

- File: `app/api/chat_gemini/route.js`
- Runtime: `edge`
- Separate Gemini proxy route with `ACCESS_KEY` gate.
- Not part of the main default frontend flow today; the browser-side Gemini integration in `service/gemini.js` is the primary active path.

## Model Definitions

### Internal Cloudflare Models

- Defined in `app/api/utils/models.js`
- Categories:
  - chat
  - gen-image
  - translate

### External Models

- Defined in `utils/models.js`
- Providers currently wired:
  - OpenAI
  - Gemini
  - DeepSeek
  - Grok
  - Ollama

## Message And Stream Handling

- Message schema helpers live in `utils/chat.js`.
- Server-side chat payload helpers live in `app/api/chat/schema.js` and `app/api/utils/index.js`.
- IDs come from `nanoid`.
- Chat context window sent to the backend is currently only the latest 4 messages plus the new message.
- Streaming server-sent event parsing for internal APIs is handled by `streamReader()`.
- OpenAI-compatible reasoning streams are normalized by `formatStreamResponse()` in `utils/thinking.js`.
- UI chunk coalescing is throttled by `utils/queue.js` to avoid excessive rerenders.
- Shared chat shape is now:
  - message: `{ role, content, reasoning }`
  - stream chunk: `{ content, reasoning, done }`
  - non-stream chat payload: `{ content, reasoning }`
- Legacy plain `text` is still returned by `GET /api/chat` for compatibility, but it mirrors `content`.

## Local Persistence

- The app depends heavily on browser localStorage through Zustand persistence.
- Third-party API keys are stored client-side in persisted state.
- Helper accessors in `utils/storage.js` read directly from localStorage.
- This means:
  - browser-only assumptions are intentional
  - secrets are not protected if entered in the client
  - SSR-safe use of these helpers must be considered carefully

## Conventions And Implementation Patterns

- Route path determines active model bucket through `mapPathToKey(path)`.
- The same `Chat` component is reused for all three tools; behavior differences should usually be implemented in `useChat(type)` and supporting components, not by duplicating full pages.
- Internal model catalog is fetched dynamically from `/api/models`.
- External provider support is provider-specific and usually added in three places:
  - `utils/models.js` for metadata
  - `service/<provider>.js` for invocation
  - `service/index.js` for routing in `getChatApi`

## Safe Places To Change Things

### Add A New Internal Cloudflare Chat/Image/Translation Model

1. Update `app/api/utils/models.js`
2. Verify the relevant route can accept the model input shape
3. Check any special response handling, especially for image models

### Add A New External Provider

1. Extend `ModelTypeEnum` and `ExternalModelHost` in `utils/models.js`
2. Add provider model entries in `utils/models.js`
3. Implement `service/<provider>.js`
4. Update `service/index.js`
5. If the provider needs extra UI/config, update model settings related components/store

### Change Chat Context Rules

- Update `genChatPostParams()` in `utils/chat.js`
- Verify streaming UI still behaves correctly with longer histories and pending messages

### Change Persisted State Shape

- Update `store/index.js`
- Consider Zustand `version` changes and migration strategy; current stores use fixed versions with no migration code

## Known Risks / Attention Points

- `service/gemini.js` contains a likely logic bug: it checks `if (genImage)` instead of `if (genImage(model))`, which makes the non-stream image-generation branch always run. Touch this carefully if working on Gemini behavior.
- External provider calls run in the browser with persisted API keys. This is convenient but not secure for multi-user/public scenarios.
- `utils/storage.js` assumes `model-store` exists and parses it directly; missing or malformed localStorage could break direct accessor calls.
- `service/ollama.js` error handling uses `res.body.json()`, which is suspicious because `body` is a stream, not a `Response`.
- Current chat context truncation to the last 4 messages is product logic, not a model limit enforced elsewhere.
- `useInitModel()` fetches only on initial mount and suppresses exhaustive-deps linting. Route changes or model refresh logic are minimal by design.

## Suggested Workflow For Future Changes

1. Identify whether the change belongs to shared chat shell, route-level API, provider integration, or persisted state.
2. Check whether the target feature uses internal Cloudflare models or external browser-side providers.
3. Trace the flow from page -> `components/chat/index.jsx` -> `hooks/useChat.js` -> `service/*` -> `app/api/*`.
4. If a change touches model metadata, update both catalog and selection assumptions.
5. If a change touches streaming, verify chunk assembly and pending-state behavior.

## Highest-Value Files To Read First

- `hooks/useChat.js`
- `store/index.js`
- `utils/chat.js`
- `service/index.js`
- `service/api.js`
- `app/api/chat/route.js`
- `app/api/gen-image/route.js`
- `app/api/trans/route.js`
- `app/api/utils/models.js`
- `utils/models.js`

---
> Source: [wuyangwang/my-chat-web](https://github.com/wuyangwang/my-chat-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
