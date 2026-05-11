## ai-draw-io-fronted

> IntelliDraw is a Next.js-based AI-powered diagram creation platform that integrates draw.io editor with AI-assisted diagram generation through natural language. It supports real-time collaboration using Yjs CRDT, team spaces, diagram marketplace, and connects to backend AI services via SSE streaming.

# IntelliDraw - AI-Powered Collaborative Diagram Platform

## Project Overview

IntelliDraw is a Next.js-based AI-powered diagram creation platform that integrates draw.io editor with AI-assisted diagram generation through natural language. It supports real-time collaboration using Yjs CRDT, team spaces, diagram marketplace, and connects to backend AI services via SSE streaming.

**Live Demo**: https://www.intellidraw.top/

## Tech Stack

| Category | Technologies |
|----------|-------------|
| Framework | Next.js 16 (App Router), React 19 |
| Language | TypeScript 5 |
| Styling | Tailwind CSS 4, Ant Design 5, Radix UI, shadcn/ui |
| State Management | Redux Toolkit |
| Collaboration | Yjs CRDT, WebSocket (@hocuspocus/provider) |
| AI Integration | Backend SSE Streaming |
| i18n | next-intl |
| Linting | Biome 2.3.8 |
| Build | Husky, lint-staged |

## Project Structure

```
├── app/                    # Next.js App Router pages
│   ├── api/chat/stream/    # AI chat SSE route handler
│   ├── diagram/            # Diagram edit/view pages
│   ├── diagram-marketplace/ # Public diagram marketplace
│   ├── my-diagrams/        # User's diagrams
│   ├── my-spaces/          # Team spaces
│   ├── my-rooms/           # Collaboration rooms
│   ├── templates/          # Diagram templates (SEO)
│   ├── solutions/          # Solution pages (SEO)
│   ├── wiki/               # Wiki pages (SEO)
│   ├── user/               # Auth pages (login/register/profile)
│   ├── admin/              # Admin dashboard
│   ├── layout.tsx          # Root layout with i18n
│   ├── providers.tsx       # Redux, Antd, Diagram context providers
│   └── globals.css         # Global styles
│
├── components/             # React components
│   ├── chat-panel.tsx      # Main AI chat panel (core component)
│   ├── simple-chat-panel.tsx # Simplified chat for demo pages
│   ├── demo-chat-panel.tsx # Demo-specific chat panel
│   ├── collaboration-panel.tsx # Real-time collaboration UI
│   ├── DrawioHome.tsx      # Main draw.io integration component
│   ├── diagram-toolbar.tsx # Diagram toolbar actions
│   ├── settings-dialog.tsx # AI provider settings dialog
│   ├── save-dialog.tsx     # Save/export dialog
│   ├── download-dialog.tsx # Download options
│   ├── file-preview-list.tsx # Uploaded file previews
│   ├── chat-input.tsx      # Chat input with file upload
│   ├── chat-message-display.tsx # Message rendering
│   ├── chat-example-panel.tsx # Example prompts
│   ├── history-dialog.tsx  # Version history
│   ├── ai-config-dialog.tsx # AI model configuration
│   ├── thinking-block.tsx  # AI reasoning/thinking display
│   ├── code-block.tsx      # Code block with syntax highlighting
│   ├── ui/                 # shadcn/ui components (button, dialog, etc.)
│   ├── admin/              # Admin management components
│   ├── room/               # Room management components
│   └── user/               # User-related components
│
├── lib/                    # Utilities and hooks
│   ├── ai-config.ts        # AI config from localStorage (user-defined)
│   ├── utils.ts            # XML validation/formatting utilities
│   ├── use-websocket-collaboration.ts # WebSocket collaboration hook
│   ├── use-yjs-collaboration.ts # Yjs collaboration hook
│   ├── yjs-collab.ts       # Yjs document management
│   ├── collab-protocol.ts  # Collaboration protocol definitions
│   ├── collab-packet.ts    # Collaboration packet handling
│   ├── cryptoUtils.ts      # Encryption for collaboration
│   ├── use-persistence.ts  # Diagram persistence hook
│   ├── use-diagram-save.ts # Diagram save logic
│   ├── use-file-processor.tsx # File upload/processing
│   ├── use-backend-chat.ts # Backend API chat integration (SSE)
│   ├── use-quota-manager.tsx # Quota management
│   ├── pdf-utils.ts        # PDF extraction utilities
│   ├── token-counter.ts    # Token counting for AI
│   ├── storage.ts          # LocalStorage utilities
│   ├── cached-responses.ts # Response caching
│   ├── request.ts          # HTTP request utilities
│   └── seo-data.ts         # SEO metadata
│
├── contexts/               # React contexts
│   ├── diagram-context.tsx # Diagram state management context
│
├── api/                    # Backend API client definitions
│   ├── index.ts            # API client initialization
│   ├── typings.d.ts        # API type definitions
│   ├── aiClientController.ts # AI client management
│   ├── diagramController.ts # Diagram CRUD
│   ├── conversionController.ts # Diagram conversion/chat history
│   ├── roomController.ts   # Collaboration room
│   ├── roomMemberController.ts # Room membership
│   ├── spaceController.ts  # Team space management
│   ├── spaceUserController.ts # Space membership
│   ├── userController.ts   # User management
│   ├── materialController.ts # Material/assets
│   ├── feedBackController.ts # User feedback
│   ├── announcementController.ts # Announcements
│   ├── systemAdminController.ts # System admin
│   ├── fileController.ts   # File upload/management
│   ├── codeParser.ts       # Code parsing API
│   ├── internalApiController.ts # Internal API
│
├── stores/                 # Redux store
│   ├── index.ts            # Store configuration
│   ├── loginUser.ts        # User state slice
│
├── config/                 # Configuration
│   ├── api.ts              # API base URL config
│   ├── menu.tsx             # Menu configuration
│
├── access/                 # Access control
│   ├── AccessLayout.tsx    # Permission check wrapper
│   ├── accessEnum.ts       # Permission enum
│   ├── checkAccess.ts      # Access check utility
│   ├── menuAccess.ts       # Menu access mapping
│
├── layouts/                # Layout components
│   ├── basiclayout/        # Basic layout wrapper
│
├── messages/               # i18n translations
│   ├── zh.json             # Chinese
│   ├── en.json             # English
│
├── styles/                 # Global styles
│   ├── markdown.css        # Markdown styling
│
├── public/                 # Static assets
│
├── instrumentation.ts      # OpenTelemetry setup (disabled - backend handles observability)
├── next.config.ts          # Next.js config (rewrites, webpack)
├── biome.json              # Biome linting config
├── tsconfig.json           # TypeScript config
├── env.example             # Environment variables template
├── openapi.config.ts       # OpenAPI generator config
├── docker-compose.yml      # Docker deployment config
└── i18n.ts                 # next-intl configuration
```

## Key Architecture Patterns

### 1. App Router with Route Handlers
- SSE route handlers at `/api/chat/stream/route.ts` and `/api/chat/custom/stream/route.ts` proxy to backend
- API rewrites to backend server (`http://localhost:8081/api` in dev)
- SEO pages: `/templates`, `/solutions`, `/wiki`

### 2. AI Integration (Backend SSE)
- **Backend handles all AI processing**: Tool calls (`edit_diagram`, `append_diagram`, `display_diagram`) processed by backend
- **SSE streaming**: `useBackendChat` hook handles Server-Sent Events from backend
- **User-defined AI config**: Optional custom baseUrl/apiKey via `ai-config-dialog.tsx`
- **System prompts**: Managed by backend, not frontend
- **Two chat panels**: `ChatPanel` (full features) and `SimpleChatPanel` (demo pages) both use `useBackendChat`

### 3. Real-time Collaboration (Yjs CRDT)
- WebSocket-based real-time sync
- End-to-end encryption using AES-GCM
- Role-based permissions (Editor/Viewer)
- Cursor position sync
- Room-based collaboration

### 4. Diagram Context (State Management)
- Centralized diagram state via React Context
- Integration with draw.io embed component
- Version history tracking
- Export/save functionality
- Collaboration state integration

### 5. Access Control
- Permission-based routing via `AccessLayout`
- Menu-level access configuration
- Role-based access: `NOT_LOGIN`, `USER`, `ADMIN`

## Development Commands

```bash
# Development (port 6002)
npm run dev

# Production build
npm run build

# Production start (port 6001)
npm run start

# Linting
npm run lint

# Format code
npm run format

# CI check
npm run check

# OpenAPI client generation
npm run openapi
```

## Environment Variables

See `env.example` for full configuration. Key variables:

### Backend Integration
- `NEXT_PUBLIC_API_BASE_URL`: Backend API URL (default: `/api` via rewrites)
- `NEXT_PUBLIC_WS_URL`: WebSocket URL for collaboration

### Optional
- `NEXT_PUBLIC_DRAWIO_BASE_URL`: Custom draw.io instance
- `ENABLE_PDF_INPUT`: Enable PDF file upload feature

## Code Conventions

### Styling
- Use Tailwind CSS classes primarily
- Ant Design for complex components (tables, forms, dialogs)
- shadcn/ui for simple UI primitives
- Radix UI for dialog, collapsible, scroll-area primitives

### Component Patterns
- Use `"use client"` directive for client components
- React Context for shared state (DiagramContext)
- Custom hooks for reusable logic (`lib/*.ts`, `lib/*.tsx`)
- Redux Toolkit for global state (loginUser)

### Import Convention
- Use `@/` alias for imports: `import { X } from "@/components/X"`
- Path alias configured in `tsconfig.json`

### Linting Rules (Biome)
- 4-space indentation
- Double quotes
- Semicolons "asNeeded" (optional)
- Many a11y rules disabled for flexibility
- `components/ui/**` excluded from linting (shadcn/ui)

### TypeScript
- Strict mode enabled
- `noEmit: true` (Next.js handles compilation)
- Target: ES2017

## AI Chat Flow

1. User sends message via `chat-input.tsx`
2. `chat-panel.tsx` uses `useBackendChat` hook for SSE streaming
3. Request sent to `/api/chat/stream` (proxied to backend)
4. Backend handles AI processing and tool calls (`edit_diagram`, `append_diagram`, `display_diagram`)
5. SSE streams AI responses and tool results back to frontend
6. XML validated and applied to draw.io via `DiagramContext`
7. Version history tracked automatically

## File Processing Flow

1. User uploads PDF/text/image via `chat-input.tsx`
2. `use-file-processor.tsx` handles file processing
3. PDFs: `pdf-utils.ts` extracts text
4. Images: uploaded to backend via `fileController.ts`
5. Content passed to AI for diagram generation

## Collaboration Flow

1. User enables collaboration via `collaboration-panel.tsx`
2. Room created/joined via `roomController.ts`
3. WebSocket connection via `use-websocket-collaboration.ts`
4. Yjs document shared via `use-yjs-collaboration.ts`
5. Changes encrypted and synced via `cryptoUtils.ts`
6. Real-time updates applied to draw.io

## Testing & Quality

- No automated tests currently (focus on production features)
- Husky pre-commit hooks with lint-staged
- Biome for linting and formatting

## Deployment

### Docker (Recommended)
```bash
docker run -d -p 6001:6001 \
  -e NEXT_PUBLIC_API_BASE_URL=/api \
  ghcr.io/wangfenghuan/w-next-ai-drawio:latest
```

Note: AI provider configuration is handled by the backend server, not the frontend.

### Manual Build
```bash
npm run build
npm run start
```

Output: `standalone` mode for Docker deployment

## Important Notes for AI Assistants

1. **Diagram XML Generation**: Backend AI generates draw.io XML format (`<mxGraphModel>...</mxGraphModel>`)
2. **Tool Calling**: Backend handles tool calls (`edit_diagram`, `append_diagram`, `display_diagram`)
3. **Streaming**: Frontend uses SSE streaming via `useBackendChat` hook
4. **Error Handling**: Backend handles retries for tool errors
5. **File Processing**: Frontend handles PDF, text, and image uploads; content sent to backend
6. **Security**: Backend validates XML and handles AI provider security

## Related Documentation

- README.md: User-facing documentation (English + Chinese)
- env.example: Environment variable reference
- biome.json: Linting configuration
- tsconfig.json: TypeScript configuration
- next.config.ts: Next.js configuration

---
> Source: [wangfenghuan/ai-draw-io-fronted](https://github.com/wangfenghuan/ai-draw-io-fronted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
