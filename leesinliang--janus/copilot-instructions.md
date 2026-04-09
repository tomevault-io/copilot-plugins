## janus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Janus** is an AI-powered GTM (Go-To-Market) OS for technical founders. It's a full-stack application with:
- **Backend**: Multi-agent system using LangChain + Google Gemini, Django REST Framework
- **Frontend**: Next.js 16 + React 19 + ReactFlow for interactive campaign visualization
- **Purpose**: Automates marketing strategy planning, A/B content generation, and metrics-driven optimization

## Repository Structure

```
├── backend/              # Django backend with multi-agent system
│   ├── src/
│   │   ├── agents/       # Multi-agent system (LangChain + Gemini)
│   │   ├── metrics/      # API endpoints for frontend polling
│   │   ├── twitter_clone/# Mock Twitter API for testing
│   │   ├── janus/        # Django project settings
│   │   └── manage.py
│   ├── demo.py           # CLI demo (strategy → content → media)
│   ├── web_demo.py       # Flask web demo
│   └── requirements.txt
│
├── frontend/janus/       # Next.js 16 frontend
│   ├── src/
│   │   ├── app/          # App router (Next.js 13+ pattern)
│   │   ├── components/   # React components (Canvas, ChatBox, etc.)
│   │   ├── services/     # API client (polling backend)
│   │   ├── hooks/        # Custom React hooks
│   │   ├── types/        # TypeScript definitions
│   │   └── utils/        # Graph parsing, layout (Dagre)
│   └── package.json
│
└── docs/                 # Auto-generated commit documentation
```

## Development Commands

### Quick Start with Makefile
```bash
# Install all dependencies and run migrations
make init

# Run both backend and frontend servers
# Note: Runs sequentially, not in parallel. Use separate terminals for both servers.
make run-backend    # Terminal 1: Django on :8000
make run-frontend   # Terminal 2: Next.js on :3000
```

### Backend Setup
```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Configure environment variables
cp .env.example .env
# Edit .env and add:
#   GOOGLE_API_KEY (required - from https://ai.google.dev/gemini-api/docs/api-key)
#   MEDIA_GEMINI_API_KEY (optional - for image/video generation)

# Run migrations
cd src && python manage.py migrate

# Start Django dev server (default: http://localhost:8000)
python manage.py runserver
```

### Frontend Setup
```bash
cd frontend/janus

# Install dependencies
npm install

# Start Next.js dev server (default: http://localhost:3000)
npm run dev

# Build for production
npm run build
npm start
```

### Docker Setup
```bash
# Quick start with Docker Compose (includes both frontend and backend)
docker-compose up --build

# Run in detached mode
docker-compose up -d --build

# View logs
docker-compose logs -f

# Stop containers
docker-compose down

# Run Django commands in container
docker-compose exec janus bash -c "cd /app/backend/src && python manage.py createsuperuser"
docker-compose exec janus bash -c "cd /app/backend/src && python manage.py migrate"

# Access services
# Frontend: http://localhost:3000
# Backend: http://localhost:8000
# Django Admin: http://localhost:8000/admin/
```

### Testing & Demos

**Backend Tests:**
```bash
# Test complete 3-scenario workflow (from backend/)
python demo.py
# Scenarios: Strategy → A/B Content → Metrics Analysis

# Test Flask web demo with UI (from backend/)
python web_demo.py
# Opens browser UI for interactive demo

# Test individual agents (from backend/src/)
cd src
python -m agents.supervisor        # Orchestrator agent
python -m agents.content_creator   # A/B content generation
python -m agents.strategy_planner  # Strategy + Mermaid diagrams
python -m agents.media_creator     # Image/video generation
python -m agents.metrics_analyzer  # Metrics analysis

# Run Django tests
python manage.py test agents
```

**Frontend Testing:**
```bash
cd frontend/janus
npm run lint   # ESLint (configured)
```

### Django Commands
```bash
cd backend/src

# Database
python manage.py makemigrations
python manage.py migrate
python manage.py showmigrations

# Admin
python manage.py createsuperuser
# Access admin at http://localhost:8000/admin/

# Shell
python manage.py shell
```

## Architecture

### Backend: 3-Layer Multi-Agent Pattern (LangChain)

**Layer 3: Orchestrator** (`agents/supervisor.py`)
- Routes user requests to specialized agents
- Sub-agents wrapped as `@tool` for LLM to call
- Pattern: `create_agent()` with sub-agents as tools

**Layer 2: Specialized Agents**
- **Strategy Planner** (`strategy_planner.py`): Generates Mermaid diagrams with 3 phases
- **Mini Strategy Agent** (`mini_strategy_agent.py`): Lightweight strategy planner variant
- **Content Creator** (`content_creator.py`): A/B content variants (text + media prompts)
- **Media Creator** (`media_creator.py`): Image generation (Imagen 3) or video generation (Veo 2)
- **Metrics Analyzer** (`metrics_analyzer.py`): Performance analysis and recommendations
- **X Platform** (`x_platform.py`): Twitter posting and scheduling operations
- **Trigger Parser** (`trigger_parser.py`): Detects triggers and conditions for automation

**Layer 1: Tools** (`agents/tools.py`)
- Low-level operations: `post_tweet`, `get_metrics`, `validate_tweet_format`
- Uses placeholder data from `agents/data/placeholder_metrics.json`

### Agent Implementation Patterns

**Structured Output Agents** (strategy_planner, content_creator):
```python
from langchain.agents import create_agent
from pydantic import BaseModel

agent = create_agent(
    model,
    tools=[],
    system_prompt="...",
    response_format=PydanticModel  # Returns validated Pydantic object
)
result = agent.invoke({"messages": [{"role": "user", "content": input_text}]})
output = result["messages"][-1]  # Pydantic object
```

**Tool-Based Agents** (x_platform, metrics_analyzer):
```python
agent = create_agent(
    model,
    tools=[tool1, tool2, tool3],
    system_prompt="..."
)
result = agent.invoke({"messages": [{"role": "user", "content": input_text}]})
output = result["messages"][-1].content  # String output
```

**Media Generation** (media_creator):
```python
# Image generation (Imagen 3)
media_agent = create_media_creator('models/gemini-2.5-flash-image')
result = media_agent.execute(prompt="...")  # Returns base64 image

# Video generation (Veo 2)
media_agent = create_media_creator('models/veo-3.1-generate-preview')
result = media_agent.execute_video(prompt="...", duration=5)  # Returns video URL
```

### Django Apps

**agents** - Multi-agent system
- Models: `Campaign`, `Post`, `ContentVariant`, `AgentMemory`, `ConversationMessage`
- API: `/api/agents/strategy/`, `/api/agents/campaigns/`
- State management: `state.py` (database-backed, migrated from in-memory)
- Admin interface: Full CRUD for all models

**metrics** - Frontend polling endpoints
- `/nodesJson/` - Returns campaign graph data (nodes + metrics)
- `/getVariants/` - Fetch A/B variants for a post
- `/selectVariant/` - Mark variant as selected
- `/createXPost/` - Publish post to Twitter clone
- `/api/approve`, `/api/graph` - Node approval/rejection workflow

**twitter_clone** - Mock Twitter API
- Mimics Twitter API v2 response format
- Models: `CloneTweet`, `CloneLike`, `CloneRetweet`, `CloneComment`
- Web UI at `/clone/` for visual testing
- Supports text, image, and video posts

**core** - Empty Django app (placeholder)

### Frontend Architecture (Next.js 16 + React 19)

**Key Components:**
- `CanvasWithPolling`: Main canvas with ReactFlow + backend polling
- `Canvas`: Static ReactFlow canvas without polling
- `TaskCardNode`: Custom node component (title, description, status, metrics)
- `ChatBox`: User input for creating campaigns
- `NodeVariantModal`: A/B variant selection modal
- `WelcomePage`: Onboarding/landing page
- `EngagementLineChart`, `EngagementPieChart`: Metrics visualization

**State Management:**
- `useGraphData` hook: Polls `/nodesJson/` every 5 seconds for updates (configurable)
- Local state for nodes/edges (ReactFlow)
- Graph diff algorithm detects changes and marks new nodes as pending approval
- API calls via `services/api.ts`

**Routing:**
- `/` - Welcome page with campaign creation
- `/canvas?campaign_id=X` - Interactive campaign editor

**Data Flow:**
1. User submits campaign via ChatBox
2. Backend generates strategy (Mermaid diagram) → saves to DB
3. Frontend polls `/nodesJson/` for updates (5s interval)
4. ReactFlow renders nodes from parsed Mermaid
5. New nodes appear with green border (pending approval)
6. User approves/rejects new nodes or clicks to view/select A/B variants
7. Backend generates content variants on approval
8. Metrics displayed in real-time

**Polling & Approval System:**
- Graph diff (`utils/graphDiff.ts`) detects new nodes and marks them `pendingApproval: true`
- Pending nodes show green border (`border-4 border-green-400`) with approve/reject buttons
- Approval is client-side only (removes `pendingApproval` flag)
- Rejection removes node and connected edges from local state
- See `frontend/janus/APPROVAL_SYSTEM.md` for detailed approval flow
- See `frontend/janus/POLLING_SYSTEM.md` for polling architecture

### Database Schema (Django ORM + SQLite)

**Campaign:**
- `campaign_id` (unique), `name`, `description`, `phase`, `status`
- `strategy` (TextField) - Stores Mermaid diagram
- `insights` (JSONField) - Marketing insights array
- `metadata` (JSONField)

**Post:**
- `post_id`, `campaign` (FK), `title`, `description`, `phase`, `status`
- `metrics` (JSONField) - Engagement data
- `next_posts` (M2M) - Graph connections

**ContentVariant:**
- `variant_id`, `post` (FK), `content`, `platform`, `selected`
- `asset` (FileField) - Image/video media
- `metadata` (JSONField) - Media prompt, generation params

**Mermaid Diagram Format:**
```
graph TB
  subgraph "Phase 1"
    NODE1[<title>Title</title><description>Desc</description>]
    NODE2[<title>Title</title><description>Desc</description>]
  end
  subgraph "Phase 2"
    NODE3[...]
  end
  subgraph "Phase 3"
    NODE4[...]
  end
  NODE1 --> NODE3
  NODE2 --> NODE3
```

Parsed by `agents/mermaid_parser.py` into nodes and connections.

## Critical Implementation Notes

### LangChain + Google Gemini

**Model Initialization:**
```python
from langchain_google_genai import ChatGoogleGenerativeAI

model = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash",  # NOT "google_genai:gemini-2.5-flash"
    temperature=0.7
)
```

**Common Pitfalls:**
1. ❌ Don't use `init_chat_model()` or `AgentExecutor` (deprecated)
2. ❌ Don't use `executor.invoke({"input": ...})` - use `agent.invoke({"messages": [...]})`
3. ❌ Don't use `JsonOutputParser` chain - use `response_format=PydanticModel`
4. ✅ Use `temperature=0` for structured output agents
5. ✅ Cast `response.content` to `str()` to avoid list type issues

### Environment Variables

**Required:**
- `GOOGLE_API_KEY` - For LangChain agents (Gemini Flash)

**Optional:**
- `MEDIA_GEMINI_API_KEY` - For Imagen 3 / Veo 2 media generation
- `NEXT_PUBLIC_API_URL` - Frontend API endpoint (default: http://localhost:8000)

### CORS Configuration

Django settings include CORS headers for local development:
```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]
```

## Common Workflows

### Creating a New Agent

1. Define Pydantic output schema (if structured output)
2. Create agent class in `backend/src/agents/`
3. Use `create_agent()` with `response_format` or `tools`
4. Add factory function to `agents/__init__.py`
5. Test with `python -m agents.your_agent`

### Adding Frontend Component

1. Create component in `frontend/janus/src/components/`
2. Import in parent component or page
3. Use TypeScript types from `types/api.ts`
4. Update `services/api.ts` if new API endpoints needed

### Extending Database Schema

1. Edit models in `backend/src/agents/models.py` or `backend/src/metrics/models.py`
2. Run `cd backend/src && python manage.py makemigrations`
3. Run `python manage.py migrate`
4. Update serializers if REST API affected (see `agents/serializers.py` or `metrics/serializers.py`)
5. Register new models in admin interface (see `agents/admin.py` or `metrics/admin.py`)

### Regenerating Campaign Strategy from Specific Phase

**New Feature:** Use the `!regenerate phase X` chat command to regenerate strategy from a specific phase onwards while preserving earlier phases.

**Chat Command:**
```
!regenerate phase 2 focus on technical demos and developer engagement
```

**How it works:**
1. Specify which phase to regenerate from (e.g., phase 2)
2. Provide new direction/prompt for the regenerated strategy
3. System archives posts from phase 2 onwards (sets `is_active=False`)
4. Strategy planner generates new phases with many-to-many connections
5. New posts are created and linked to existing phase 1 posts
6. Content generation runs automatically for new posts only

**Key Features:**
- **Preserves Earlier Phases**: Phase 1 posts remain unchanged with their connections intact
- **Many-to-Many Connections**: Supports branching (1→N) and merging (N→1) between old and new posts
- **Version Tracking**: Campaign tracks `current_version`, posts track `version` and `is_active` status
- **Smart Linking**: Strategy planner determines connections based on content relevance
- **Dynamic Node Count**: Generates 2-5 nodes per phase (not fixed at 3)

**Example Workflow:**
```bash
# In ChatBox on canvas page:
!regenerate phase 2 pivot to developer-first strategy with technical demos

# System will:
# 1. Keep Phase 1 posts active
# 2. Archive old Phase 2 & 3 posts (is_active=False)
# 3. Generate new Phase 2 & 3 posts
# 4. Connect Phase 1 posts to new Phase 2 posts
# 5. Generate A/B content for new posts
```

**API Endpoint:**
```bash
POST /api/agents/regenerate-strategy/
{
  "campaign_id": "campaign_1",
  "phase_num": 2,
  "new_direction": "Focus on ProductHunt launch with developer communities"
}
```

**Database Changes:**
- `Campaign.current_version` (IntegerField) - tracks strategy version
- `Post.version` (IntegerField) - tracks which version this post belongs to
- `Post.is_active` (BooleanField) - filters archived posts from display

**Note:** Archived posts are kept in the database for future versioning UI but hidden from display.

### Debugging Common Issues

**Backend:**
```bash
# Check Django logs for errors
cd backend/src && python manage.py runserver --verbosity 3

# Test individual agents
python -m agents.strategy_planner
python -m agents.content_creator

# Check database state
python manage.py shell
>>> from agents.models import Campaign, Post
>>> Campaign.objects.all()

# View admin interface
# Create superuser first: python manage.py createsuperuser
# Then visit http://localhost:8000/admin/
```

**Frontend:**
```bash
# Check browser console for errors (Chrome DevTools)
# Enable verbose logging in CanvasWithPolling.tsx by adding console.logs

# Test API endpoints directly
curl http://localhost:8000/nodesJson/
curl http://localhost:8000/getVariants/ -X POST -H "Content-Type: application/json" -d '{"post_id": 1}'

# Check Next.js build errors
npm run build
```

**CORS Issues:**
- Ensure Django `CORS_ALLOWED_ORIGINS` includes frontend URL
- Check `backend/src/janus/settings.py` for CORS configuration
- Verify `NEXT_PUBLIC_API_URL` in `.env.local` matches backend URL

## Testing Without Real APIs

- **Twitter API**: Use `twitter_clone` app at `/clone/` for posting and metrics
- **Metrics**: Placeholder data in `agents/data/placeholder_metrics.json`
- **Media**: Demo scripts save to `backend/generated_videos/`

## Performance Notes

- Frontend polls backend every 5 seconds (`useGraphData`, configurable in `CanvasWithPolling.tsx`)
- Backend returns `changes: false` when no updates to avoid unnecessary graph rebuilds
- Database queries optimized with `.select_related()` and `.prefetch_related()`
- Media files stored in `backend/src/media/` (served by Django in DEBUG mode)
- ReactFlow layout uses Dagre for auto-positioning
- Graph diff algorithm only updates changed nodes, not entire graph

## Known Limitations

- SQLite database (migrate to PostgreSQL for production)
- No real Twitter API integration yet (uses mock clone)
- Video generation is slow (~30-60 seconds per video)
- No user authentication (single-user system currently)

## Important Implementation Details

### A/B Metrics Architecture (Migration Planned)
Current implementation stores single metrics per post. Migration plan exists to support:
- Both A and B variants posted simultaneously with separate `tweet_id`s
- Separate metrics tracking for each variant
- Display modes: max (canvas nodes), aggregated (charts), separate (modal)
- See `AB_METRICS_MIGRATION_PLAN.md` for full migration strategy

### Graph Diff Algorithm
Located in `frontend/janus/src/utils/graphDiff.ts`:
- Compares previous and current graph states
- Detects added/removed/updated nodes and edges
- Marks new nodes as `pendingApproval: true`
- Only triggers ReactFlow updates when `changes: true` from backend

### Mermaid Parser
Located in `frontend/janus/src/utils/mermaidParser.ts` and `backend/src/agents/mermaid_parser.py`:
- Parses custom Mermaid-like syntax with embedded XML tags
- Frontend parser: Converts to ReactFlow nodes/edges with Dagre layout
- Backend parser: Validates and extracts structured data from diagrams
- Node format: `NODE_ID[<title>Title</title><description>Desc</description>]`

### State Management Migration
The project migrated from in-memory state to database-backed state:
- Old: `agents/state.py` with in-memory dictionaries
- New: Django ORM models (`Campaign`, `Post`, `ContentVariant`, etc.)
- `AgentMemory` model stores conversation history
- All agent operations now persist to SQLite

## Future Enhancements

From `backend/notes/janus.md`:
- Real X/Twitter API integration (currently uses mock `twitter_clone`)
- ProductHunt API integration
- Trigger detection engine (auto-swap underperforming variants)
- Vector DB for semantic campaign search
- PostgreSQL for production (currently SQLite)
- ReactFlow canvas for visual campaign editing (currently read-only with approval/rejection)
- User authentication and multi-tenancy
- WebSocket support for real-time updates (instead of polling)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeeSinLiang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LeeSinLiang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
