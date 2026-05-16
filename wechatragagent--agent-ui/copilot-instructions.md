## agent-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 沟通规范

- 所有内容必须使用 **中文** 交流（包括代码注释），但是文案与错误提示要使用英文。
- 遇到不清楚的内容应立即向用户提问。
- 表达清晰、简洁、技术准确。
- 在代码中应添加必要的注释解释关键逻辑。

## Commands

### Development
- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

### Docker
- `docker build -t agent-ui .` - Build Docker image
- `docker run -p 3000:3000 agent-ui` - Run container (standalone)
- `docker-compose up` - Start frontend only (requires backend running)
- `docker-compose up --build` - Rebuild and start frontend
- `docker-compose up` - Start frontend container in wechat-rag network

### TypeScript
- Type checking is configured via `tsconfig.json`
- Uses strict TypeScript with Next.js plugin
- Path alias `@/*` maps to root directory for imports
- No test framework currently configured

### Environment Variables
- `WECHAT_API_URL` - WeChat API endpoint (default: http://127.0.0.1:5030)
- `BACKEND_API_URL` - Backend service endpoint (default: http://127.0.0.1:8080)
- `BACKEND_URL` - Alternative backend URL used in API routes (default: http://127.0.0.1:8080)
- `OPENROUTER_API_KEY` - OpenRouter API key for LLM services

**Note**: Runtime environment variables are provided to client-side components via `/api/config` endpoint. This allows containers to read Docker environment variables at runtime rather than build time.

## Architecture

This is a WeChat RAG (Retrieval-Augmented Generation) chatbot frontend built with Next.js 15, React 19, and TypeScript. The application provides a UI for interacting with WeChat chat history through an AI-powered chat interface.

### Core Architecture Patterns

**State Management**: Centralized context-based architecture using `AppProvider` in `contexts/app-context.tsx`. All major application state (settings, UI state, chat, vectorization, WeChat) is managed through this provider.

**Hook-Based Services**: Business logic is encapsulated in custom hooks:
- `use-chat.ts` - Chat functionality with streaming responses via SSE
- `use-vectorization.ts` - Background vectorization progress tracking
- `use-wechat.ts` - WeChat API integration
- `use-settings.ts` - Persistent settings management via localStorage
- `use-mobile.ts` - Mobile device detection

**Component Structure**:
- `components/setup/` - Setup wizard components
- `components/chat/` - Chat interface and message handling  
- `components/sync/` - Vectorization/sync progress monitoring
- `components/help/` - FAQ and help components
- `components/ui/` - Reusable UI components (shadcn/ui based)

**API Integration**: RESTful API layer in `lib/api/` with separate modules for:
- `chat.ts` - OpenRouter API for LLM interactions
- `vectorization.ts` - Background processing with SSE progress updates
- `wechat.ts` - WeChat data retrieval and chatroom management
- `base.ts` - Shared HTTP client utilities

### Key Data Flow

1. **Setup Flow**: Users configure API endpoints and credentials through setup wizard
2. **Chat Flow**: Messages are sent to backend, which retrieves relevant WeChat context and forwards to LLM
3. **Vectorization Flow**: Background processing of WeChat data with real-time progress updates via SSE
4. **Persistence**: Settings stored in localStorage, conversations cached locally

### Technology Stack

- **UI Framework**: Next.js 15 with App Router, React 19
- **Styling**: Tailwind CSS v4 with shadcn/ui components and tw-animate-css
- **State**: React Context + custom hooks
- **Forms**: React Hook Form with Zod validation
- **HTTP**: Fetch API with streaming support
- **Icons**: Lucide React
- **Markdown**: react-markdown with remark-gfm for GFM support
- **Notifications**: Sonner for toast notifications
- **Themes**: next-themes for dark/light mode support

### File Organization

- `/app` - Next.js app router pages, layouts, and API routes
- `/components` - React components organized by feature (chat, setup, sync, help, ui)
- `/contexts` - React context providers (centralized app state)
- `/hooks` - Custom React hooks for business logic
- `/lib` - Utilities, types, API clients, and shared constants
- `/doc` - Chinese documentation for the project

The application supports Chinese language and is specifically designed for WeChat chat analysis and interaction.

## Development Patterns

### Code Style
- Use TypeScript with strict mode enabled
- Follow React 19 patterns with hooks and functional components
- Prefer functional programming patterns over class-based components
- Use Zod for runtime type validation in forms
- Event handling with Server-Sent Events (SSE) for real-time updates

### State Management Pattern
- All major state flows through the `AppProvider` context
- Hooks encapsulate business logic and API interactions
- Settings persist to localStorage automatically
- Conversations are cached locally with auto-save functionality

### API Communication
- RESTful endpoints for standard operations
- Server-Sent Events (SSE) for streaming chat responses and progress updates
- Error handling is centralized through context providers
- Connection testing utilities available in `base.ts`

### Component Guidelines
- UI components from shadcn/ui library in `components/ui/`
- Feature components organized by domain (setup, chat, sync, default)
- Form handling with React Hook Form + Zod validation
- Responsive design with mobile detection via `use-mobile.ts` hook

## Critical Architectural Understanding

### Context Provider Architecture
The `AppProvider` (`contexts/app-context.tsx:44`) serves as the single source of truth, managing:
- Settings via `use-settings.ts` hook with localStorage persistence
- Chat state via `use-chat.ts` with EventSource streaming
- UI state including view routing (`default` | `setup` | `chat` | `sync`)
- Conversation history with auto-save to localStorage

### View-Based Routing
Main routing logic in `app/page.tsx:12` switches between views:
- `default` - Welcome/landing page with navigation
- `setup` - Configuration wizard for API endpoints and credentials
- `chat` - Main chat interface with streaming responses
- `sync` - Vectorization progress monitoring and control

### Service Layer Pattern
API services in `lib/api/` follow consistent patterns:
- `ApiClient` base class in `base.ts` provides common HTTP methods
- Service-specific classes extend this pattern
- Connection testing methods standardized across services
- SSE streaming handled specifically in `ChatService`

### Data Persistence Strategy
- Settings: localStorage via `use-settings.ts` with key `wechat-rag-settings`
- Conversations: localStorage via app context with key `wechat-rag-conversations`
- Auto-serialization/deserialization of Date objects for conversation timestamps

### Error Handling Pattern
Centralized error handling through:
- Context provider error state management
- Service-level error propagation to onError callbacks
- Connection status tracking for each external service

### Streaming Implementation
Chat streaming via Server-Sent Events:
- `ChatService.createChatStream()` creates EventSource connections
- Real-time message assembly in `use-chat.ts:133`
- Graceful connection cleanup and error recovery

## Testing

Currently no test framework is configured. If user requests testing:
- Check for existing test setup in `package.json` before suggesting frameworks
- Common patterns would be Jest with React Testing Library for this tech stack
- Consider the existing TypeScript and Next.js 15 configuration when setting up tests

## Backend Integration

The frontend connects to a separate backend service:
- Backend URL defaults to `http://127.0.0.1:8080` (configurable via `BACKEND_API_URL` env var)
- WeChat API URL defaults to `http://127.0.0.1:5030` (configurable via `WECHAT_API_URL` env var)
- Chat endpoint: `GET /api/chat` with query parameters (question, modelName, apiKey)
- Uses Server-Sent Events (SSE) for streaming responses
- Groups and members parameters are logged but not yet supported by backend

## Next.js Configuration

Development configuration in `next.config.ts`:
- Minimal configuration with environment variable injection
- Application version exposed via `NEXT_PUBLIC_APP_VERSION` from package.json
- API proxying handled through dedicated route handlers in `/app/api/` directory

## Proxy API Pattern

API route handlers in `/app/api/` directory provide backend integration:
- `/api/config` - Runtime environment variable configuration
- `/api/proxy/backend/chat/route.ts` - Chat endpoint with SSE streaming support
- `/api/proxy/wechat/[...path]/route.ts` - WeChat API proxy with dynamic path handling
- This pattern eliminates CORS issues and provides unified API interface
- All external service communication is proxied through these endpoints

## Form Validation Strategy

Forms use React Hook Form with Zod validation:
- Schema definitions should be co-located with form components
- Use `@hookform/resolvers/zod` for seamless integration
- Error messages should be in English as per project requirements
- Form state management follows the hook-based pattern

## Event Source (SSE) Implementation Details

Real-time communication via Server-Sent Events:
- `EventSource` connections managed in hooks like `use-chat.ts`
- Connection cleanup handled in `useEffect` cleanup functions
- Graceful error handling and reconnection logic built into hooks
- Message parsing assumes JSON format for structured data

## Mobile Responsiveness

Mobile detection via `use-mobile.ts` hook:
- Uses `window.matchMedia` to detect screen size
- Components conditionally render mobile vs desktop layouts
- Sidebar behavior adapts based on mobile state
- Touch-friendly UI elements on mobile devices

## Development Environment

When running `npm run dev`:
- Uses Next.js 15 with Turbopack for faster development builds
- Hot reload is configured for all file types
- Development server runs on http://localhost:3000 by default
- TypeScript compilation happens in parallel with the dev server

## Docker Deployment

### Deployment Options

#### 2. Frontend Only
如果后端已经在运行，只启动前端：
```bash
# 确保后端服务在 wechat-rag 网络中运行
docker-compose up --build
```

#### 3. Standalone
独立运行前端容器：
```bash
docker build -t agent-ui .
docker run -p 3000:3000 \
  -e WECHAT_API_URL=http://host.docker.internal:5030 \
  -e BACKEND_API_URL=http://host.docker.internal:8080 \
  agent-ui
```

### Environment Configuration
Configure these environment variables in `docker-compose.yml` or pass to `docker run`:
- `WECHAT_API_URL` - WeChat API service endpoint
- `BACKEND_API_URL` - Backend service endpoint  
- `BACKEND_URL` - Alternative backend URL for API routes
- `OPENROUTER_API_KEY` - OpenRouter API key (optional, can be configured in UI)

### Network Architecture
- All services run in the `wechat-rag` Docker network
- Frontend communicates with backend using service name: `agent-web:8080`
- WeChat API uses `host.docker.internal:5030` (external to Docker)
- Services can communicate directly without exposing all ports

### Service Dependencies
```
agent-ui (port 3000)
  ↓ depends on
agent-web (port 8080)
  ↓ depends on
elasticsearch (port 9200) + redis (port 6379)
```

### File Structure
- `docker-compose.yml` - Frontend only deployment (connects to wechat-rag network)
- `next.config.js` - Production configuration (JavaScript)
- `next.config.ts` - Development configuration (TypeScript)

### Production Optimizations
- Uses Next.js standalone output mode for smaller Docker images
- Separate JavaScript config file eliminates TypeScript runtime dependency
- Multi-stage Docker build with build-time dependency installation
- Runtime environment variable support via API endpoints

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Source: [WechatRagAgent/agent-ui](https://github.com/WechatRagAgent/agent-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
