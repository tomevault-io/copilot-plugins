## derlg

> **DerLg** is a comprehensive Cambodian travel booking platform featuring a mobile-first web application, a robust backend services layer, and an agentic AI concierge. The project is designed to facilitate trip discovery, hotel and transportation bookings, and emergency support for travelers in Cambodia, with multi-language support (English, Khmer, Chinese).

# GEMINI.md - DerLg Project Context

## Project Overview
**DerLg** is a comprehensive Cambodian travel booking platform featuring a mobile-first web application, a robust backend services layer, and an agentic AI concierge. The project is designed to facilitate trip discovery, hotel and transportation bookings, and emergency support for travelers in Cambodia, with multi-language support (English, Khmer, Chinese).

### Architecture
- **Backend (`/backend`):** NestJS framework with Prisma ORM, Supabase (PostgreSQL), Redis (caching/sessions), and Stripe (payments). It serves as the single source of truth and provides REST APIs for the frontend and dedicated tool endpoints for the AI agent.
- **Frontend (`/frontend`):** Next.js 14 (App Router) mobile-first PWA. Uses Tailwind CSS, shadcn/ui, Zustand for state management, and React Query for data fetching. Features full offline capability for maps and cached place information.
- **AI Agent (`/llm_agentic_chatbot`):** Python (FastAPI) conversational concierge using LangGraph and Claude Sonnet 4.5. It communicates with the frontend via WebSockets and executes 20 specialized tools against the NestJS backend to assist users with bookings and information.

---

## Building and Running

### Prerequisites
- Node.js 20+
- Python 3.11+
- Docker & Docker Compose
- Redis (Local or Upstash)
- Supabase Account
- Stripe API Keys

### Backend (NestJS)
```bash
cd backend
npm install
npm run start:dev        # Development mode
npm run test             # Unit tests
npm run test:e2e         # E2E tests
npx prisma migrate dev   # Database migrations
```

### Frontend (Next.js)
```bash
cd frontend
npm install
npm run dev             # Starts dev server on http://localhost:3000
npm run build           # Production build
```

### AI Agent (Python)
```bash
cd llm_agentic_chatbot
pip install -r requirements.txt
uvicorn main:app --reload --port 8000  # Development mode
pytest                  # Run tests (use SKIP_SETTINGS_INIT=1)
```

### Full Stack (Docker)
A `docker-compose.yml` is available in the root (or `llm_agentic_chatbot`) to orchestrate the services.
```bash
docker-compose up --build
```

---

## Development Conventions

### General
- **Multi-language Support:** Content must support EN (English), KH (Khmer), and ZH (Chinese).
- **Security:** Use JWT for authentication. AI tools are secured via `X-Service-Key`. Never log secrets.
- **Testing:** Prioritize property-based testing (using `fast-check` in NestJS and `hypothesis` in Python) to validate correctness properties.

### Backend (NestJS)
- **Modular Design:** 16+ feature modules (Auth, Bookings, Payments, Emergency, etc.).
- **Prisma:** Use Prisma for type-safe DB access. Define all models in `schema.prisma`.
- **Validation:** Use `class-validator` and `zod` for input and environment validation.
- **Logging:** Structured logging with Winston and Sentry for error tracking.

### Frontend (Next.js)
- **Mobile-First:** Design components primarily for mobile viewports.
- **State:** Use Zustand for global UI state and React Query for server-side state.
- **Components:** Utilize shadcn/ui components customized with the DerLg design system.

### AI Agent (Python)
- **LangGraph:** Conversation flow is managed by a state machine (Discovery -> Suggestion -> ... -> Payment).
- **Tools:** Agents use 20 tools grouped by category (Discovery, Customization, Booking, Payment, Info).
- **Real-time:** Use WebSockets for chat and Redis Pub/Sub for payment event notifications.

---

## Key Directories
- `docs/`: Comprehensive documentation including implementation plans, task lists, and architectural designs.
- `.kiro/`: MCP (Model Context Protocol) settings and agentic specifications.
- `.opencode/`: Specialized agent definitions (BackendDev, DBArchitect, etc.).
- `backend/prisma/`: Database schema and migrations.
- `frontend/app/`: Next.js App Router structure.
- `llm_agentic_chatbot/agent/`: Core AI agent logic and LangGraph definition.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Choeng-Rayu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
