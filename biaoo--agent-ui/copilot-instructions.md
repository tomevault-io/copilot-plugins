## agent-ui

> - Install deps: `./scripts/setup.sh`

# AGENTS.md

## Setup commands

### Quick start

- Install deps: `./scripts/setup.sh`
- Start dev servers: `./start-dev.sh`
- Run tests: `./test.sh`

### Manual setup

- Backend deps: `cd backend && uv pip install -e .`
- Frontend deps: `cd frontend && npm install`
- Backend dev server: `cd backend && app-dev` (with auto-reload) or `cd backend && app` (production)
- Frontend dev server: `cd frontend && npm run dev`

## Project overview

This is a modern AI Agent application built with:

- **Backend**: Python 3.12 + Agno framework + FastAPI
- **Frontend**: TypeScript + Next.js 16 + React 19 + Tailwind CSS 4 + CopilotKit
- **Communication**: AG-UI Protocol for real-time agent interactions via CopilotKit
- **Architecture**: Monorepo with backend and frontend integration

## Code style

### Backend (Python)

- Use Python 3.12+ with strict type hints
- Follow PEP 8 style guide
- Use `uv` for package management
- Docstrings follow Google style
- Async/await for I/O operations
- Environment variables for configuration

### Frontend (TypeScript)

- TypeScript strict mode enabled
- Next.js 16 App Router architecture
- Use functional React components with hooks
- Tailwind CSS 4 for styling
- CopilotKit for AI agent integration
- AG-UI protocol via @ag-ui/agno adapter
- Server components by default, client components when needed

### General

- Conventional commits for git messages
- Write tests for all new features
- Use descriptive variable and function names
- Keep components small and focused
- Error boundaries for graceful failure handling

## Dev environment tips

### Backend development

- Use `uv venv --python 3.12` for virtual environments
- Run `uv pip install -e .` for editable installs
- Check `.env.example` for required environment variables
- Use `app-dev` for development server (auto-reload) or `app` for production server
- Logs are structured JSON for easy parsing

### Frontend development

- Use `npm` as the package manager
- Run `npm run lint` for linting with ESLint
- Hot reload enabled on file changes (Next.js Fast Refresh)
- CopilotKit runtime handles AG-UI protocol connections
- API routes in `app/api/copilotkit/route.ts` for backend integration

### Testing

- Backend tests: `cd backend && python -m pytest tests/ -v`
- Frontend: Testing setup pending (to be configured)
- Integration tests: `./test.sh`
- Coverage reports generated automatically

### Common workflows

- Add new agent: Create in `backend/src/agents/` (see `backend/AGENTS.md`)
- Add new tool: Create in `backend/src/tools/` (see `backend/AGENTS.md`)
- Add new page: Create in `frontend/app/`
- Add API route: Create in `frontend/app/api/`
- Update CopilotKit integration: Modify `frontend/app/api/copilotkit/route.ts`

## Testing instructions

### Prerequisites

- Backend: `cd backend && source .venv/bin/activate`
- Frontend: Node.js 20+ and npm installed

### Running tests

- Backend only: `cd backend && python -m pytest tests/ -v`
- Frontend: Testing framework to be configured
- Integration tests: Manual testing via running both servers

### Test coverage

- Backend: Generates HTML coverage report in `backend/htmlcov/`
- Frontend: To be configured

### Test patterns

- Unit tests for individual functions/classes
- Integration tests for API endpoints
- Agent integration tests for AG-UI protocol communication

## Security considerations

### API security

- All API endpoints validate input with Pydantic models
- Rate limiting implemented on chat endpoints
- CORS configured for frontend domain only
- Environment variables for all secrets (no hardcoded keys)

### Frontend security

- Next.js built-in security features enabled
- XSS protection through React's built-in escaping
- API keys stored in environment variables, not client code
- CopilotKit runtime handles secure agent communication
- Server-side API routes for backend integration

### Data protection

- No sensitive data logged in production
- User conversations stored encrypted
- Regular security audits of dependencies
- GDPR compliance for user data handling

## Performance considerations

### Backend optimization

- Async request handling with FastAPI
- Connection pooling for database operations
- Caching for frequently accessed knowledge
- Streaming responses for real-time chat

### Frontend optimization

- Automatic code splitting with Next.js App Router
- Server components for reduced client bundle size
- Lazy loading for large components
- React.memo for expensive renders
- Image optimization with next/image
- Font optimization with next/font

### Monitoring

- Response time tracking for all API calls
- Memory usage monitoring for agent instances
- Frontend bundle size analysis
- Database query performance logging

## Common workflows

- Add new agent: Create in `backend/src/agents/` (see `backend/AGENTS.md`)
- Add new tool: Create in `backend/src/tools/` (see `backend/AGENTS.md`)
- Add new page: Create in `frontend/app/` (see `frontend/AGENTS.md`)
- Add API route: Create in `frontend/app/api/` (see `frontend/AGENTS.md`)
- Update CopilotKit integration: Modify `frontend/app/api/copilotkit/route.ts`

## Sub-Level Documentation Standards

### Backend Documentation (`backend/AGENTS.md`)

The backend follows these additional guidelines:

- Use `uv` for package management (not `pip`)
- Server commands: `app-dev` (development), `app` (production)
- Python 3.12+ with strict type hints
- Qwen models require role mapping fix
- Tool development standards in `backend/src/tools/AGENTS.md`
- Agent development standards in `backend/src/agents/AGENTS.md`

### Frontend Documentation (`frontend/AGENTS.md`)

The frontend follows these additional guidelines:

- Use `npm` as package manager
- Next.js 16 App Router architecture
- TypeScript strict mode enabled
- Server Components by default, Client Components when needed
- CopilotKit for AI agent integration
- AG-UI protocol via @ag-ui/agno adapter

### Agent Development (`backend/src/agents/AGENTS.md`)

When creating agents:

- Use folder-based structure: `agent_name/__init__.py`, `agent.py`, `README.md`
- Choose model based on complexity: qwen-plus (simple), qwen-max (complex)
- Include detailed `instructions` with domain knowledge
- Document HITL (Human-in-the-Loop) requirements
- Follow naming conventions and import order

### Tool Development (`backend/src/tools/AGENTS.md`)

When creating tools:

- Separate core logic (`core.py`) from tool wrapper (`__init__.py`)
- Use complete type hints and Google-style docstrings
- Test core functions directly, not via `.entrypoint`
- Document HITL requirements clearly
- Use `@tool` decorator with detailed `instructions`

## Resources

- Backend details: See `backend/AGENTS.md`
- Frontend details: See `frontend/AGENTS.md`
- Agent development: See `backend/src/agents/AGENTS.md`
- Tool development: See `backend/src/tools/AGENTS.md`
- [Agno documentation](https://docs.agno.com)
- [AG-UI protocol documentation](https://docs.ag-ui.com)
- [Next.js documentation](https://nextjs.org/docs)
- [CopilotKit documentation](https://docs.copilotkit.ai)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Biaoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Biaoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
