## wp-agent-template

> **Name:** AI Chatbot Lite

# AI Chatbot Lite - Cursor Rules

## Project Overview

**Name:** AI Chatbot Lite
**Description:** Lightweight Gemini-powered customer support AI assistant with WhatsApp Cloud API integration. The bot maintains conversation history through Gemini's native chat sessions and uses a Markdown knowledge base for grounded responses—no vector databases or heavy RAG stack required.

**Stack:**
- Language: Python 3.11+
- AI: Google Gemini SDK
- Web Framework: FastAPI + Uvicorn
- WhatsApp Integration: PyWa library
- Package Manager: uv
- Deployment: Google Cloud Run (Docker containers)

**Key Features:**
- Multi-language support (12+ languages)
- Markdown-based knowledge base (no vector DB needed)
- Optional WhatsApp integration (gracefully disabled if credentials missing)
- Session-based conversation history
- Health monitoring endpoints

---

## ⚠️ CRITICAL AI ASSISTANT RULES ⚠️

### NEVER CREATE DOCUMENTATION FILES

**ABSOLUTELY FORBIDDEN:**
- ❌ NEVER create .md files (README.md, GUIDE.md, DEPLOYMENT.md, etc.)
- ❌ NEVER create documentation files in any format unless EXPLICITLY requested by user
- ❌ NEVER create summary files, guide files, or reference files
- ❌ NEVER suggest creating documentation files

**INSTEAD:**
- ✅ Provide all information directly in the chat response
- ✅ Answer questions thoroughly inline
- ✅ Use code blocks and formatting in chat
- ✅ Keep all project documentation in THIS .cursorrules file only

**EXCEPTION:**
- Only create documentation files if the user explicitly says: "create a file named X.md" or similar direct request

---

### RELY ON CLOUD RUN AND DEPLOYMENT SERVICES (DON'T BUILD FROM SCRATCH)

**CRITICAL PRINCIPLE:**
- ❌ NEVER implement rate limiting, semaphores, concurrency controls, or load balancing from scratch
- ❌ NEVER build infrastructure features that Cloud Run already provides
- ❌ NEVER over-engineer solutions when the deployment platform handles it

**INSTEAD:**
- ✅ Use Cloud Run's built-in features:
  - Concurrency limits (configured in cloudbuild.yaml: `--concurrency`)
  - Autoscaling (min/max instances)
  - Request queuing and distribution
  - Load balancing (automatic)
  - Health checks (automatic)
- ✅ Configure these in `cloudbuild.yaml` or Cloud Run settings, NOT in application code
- ✅ Keep application code simple - handle business logic only
- ✅ Let Cloud Run handle infrastructure concerns

**REMEMBER:**
- We are users of deployment services, not building infrastructure
- Cloud Run is a managed service - use its features
- Keep code simple and focused on business logic

---

## System Architecture

### Component Structure
```
src/aiChatbot/
├── adapters/            # Channel adapters (WhatsApp via PyWa)
│   ├── pywaWhatsappAdapter.py   # PyWa-based WhatsApp adapter (active)
│   └── whatsappAdapter.py       # Legacy adapter (kept for rollback)
├── api/                 # FastAPI application
│   └── app.py          # Webhook endpoints + health checks
├── interfaces/         # Abstract base classes
│   ├── aiService.py
│   ├── channelAdapter.py
│   ├── embeddingService.py
│   └── messageProcessor.py
├── models/             # Data models
│   ├── botConfig.py    # Configuration loader
│   ├── chatSession.py  # Session state
│   └── standardMessage.py  # Unified message format
├── services/           # Core business logic
│   ├── channelManager.py        # Channel registration & routing
│   ├── geminiAIService.py       # Gemini SDK integration
│   ├── messageProcessorService.py  # Message handling pipeline
│   ├── serviceFactory.py        # Dependency injection
│   └── sessionManager.py        # Conversation state management
├── utils/              # Utilities
│   ├── languageDetector.py  # Automatic language detection
│   └── promptManager.py     # Centralized prompt management
├── main.py            # CLI entry point
└── asgi.py            # ASGI application for Cloud Run
```

### Message Flow
1. **Incoming Webhook** → FastAPI endpoint (`/webhook/whatsapp`)
2. **Channel Adapter** → Converts WhatsApp message to StandardMessage
3. **Message Processor** → Routes to AI service
4. **AI Service** → Gemini generates response with knowledge base context
5. **Channel Adapter** → Sends response back via WhatsApp Cloud API

### Configuration System
- **Environment Variables:** `.env` for local, Google Secret Manager for production
- **Prompts:** `data/prompts.json` (centralized, hot-reloadable)
- **Knowledge Base:** `data/knowledge-base.md` (simple Markdown file)
- **Bot Config:** `src/aiChatbot/models/botConfig.py` (loads from env)

---

## Cloud Run Deployment

### Required Secrets (Google Secret Manager)

Store these in Secret Manager (never commit to repo):
```
GEMINI_API_KEY              # Google AI Studio / Vertex API key
WHATSAPP_PHONE_NUMBER_ID    # WhatsApp Cloud API phone number ID (optional)
WHATSAPP_ACCESS_TOKEN       # WhatsApp permanent access token (optional)
WHATSAPP_WEBHOOK_VERIFY_TOKEN  # Token for webhook verification (optional)
```

### Deployment Commands

**Manual Deployment:**
```bash
# Deploy using script
./deploy-cloudrun.sh production

# Or deploy directly
gcloud run deploy your-service-name \
  --region europe-west1 \
  --source .
```

---

## Development Guidelines

### Setting Up Local Environment

```bash
# 1. Install uv package manager
pip install uv

# 2. Install dependencies
uv pip install -e ".[dev]"

# 3. Create .env file
cp .env.example .env
# Edit .env with your API keys

# 4. Run locally
uv run python -m aiChatbot.main
# Server runs on http://localhost:8000
```

### Project Standards

**Python Version:** 3.11+

**Code Style:**
- Formatter: Black (line length: 88)
- Import sorting: isort (Black-compatible profile)
- Type checking: mypy (strict mode)
- Linting: ruff

**File Naming:**
- Modules: `camelCase.py` (e.g., `sessionManager.py`)
- Classes: `PascalCase` (e.g., `SessionManager`)
- Functions/Methods: `camelCase` (e.g., `processMessage`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`)

**Async Conventions:**
- Use `async`/`await` for I/O operations (API calls, file operations)
- Service methods that make external calls should be `async`
- Adapters use `async` for sending messages

**Error Handling:**
- Use try/except blocks for external API calls
- Log errors with context (user ID, message ID, etc.)
- Return graceful error messages to users
- Never expose internal errors to end users

### Configuration Management

**Prompt Updates:**
Edit `data/prompts.json` to update AI behavior without code changes.

**Knowledge Base Updates:**
Edit `data/knowledge-base.md` to add/update company information. Content is injected into Gemini's system prompt.

### Testing Requirements

**Run Tests:**
```bash
# All tests
uv run pytest

# With coverage
uv run pytest --cov=aiChatbot tests/
```

**Pre-Commit Checks:**
```bash
# Format code
black src/ tests/

# Sort imports
isort src/ tests/

# Type check
mypy src/

# Lint
ruff check src/ tests/
```

---

## Code Patterns & Best Practices

### Dependency Injection via ServiceFactory

Always use `ServiceFactory` for creating services:
```python
from aiChatbot.services.serviceFactory import serviceFactory

# Get AI service
ai_service = await serviceFactory.getAIService()

# Get message processor
message_processor = serviceFactory.getMessageProcessor()
```

### Channel Adapter Pattern

When adding new channels, implement `ChannelAdapter` interface:
```python
from aiChatbot.interfaces.channelAdapter import ChannelAdapter
from aiChatbot.models.standardMessage import StandardMessage

class MyAdapter(ChannelAdapter):
    async def initializeChannel(self) -> None:
        # Setup webhook/polling
        pass
    
    def receiveMessage(self, payload: Dict) -> StandardMessage:
        # Convert to StandardMessage
        pass
    
    async def sendMessage(self, content: str, channelId: str) -> bool:
        # Send message
        pass
```

### Language Detection

Language is auto-detected per message:
```python
from aiChatbot.utils.languageDetector import detectLanguage

lang = detectLanguage("Merhaba")  # Returns "tr"
lang = detectLanguage("Hello")    # Returns "en"
```

---

## WhatsApp Integration (PyWa)

### Configuration

```bash
# Required
WHATSAPP_PHONE_NUMBER_ID=your_phone_number_id
WHATSAPP_ACCESS_TOKEN=your_access_token
WHATSAPP_WEBHOOK_VERIFY_TOKEN=your_verify_token
```

**Graceful Degradation:**
If WhatsApp credentials are missing, the adapter is disabled automatically. The bot runs in "Gemini-only" mode.

### Webhook Setup

After deployment, configure webhook in Meta Developer Console:
- Webhook URL: `https://your-service.run.app/webhook/whatsapp`
- Verify Token: (value of `WHATSAPP_WEBHOOK_VERIFY_TOKEN`)
- Subscribe to: `messages` events

---

## Monitoring & Debugging

### Health Checks

**Endpoint:** `GET /health`
**Returns:** `{"status": "healthy", "timestamp": "..."}`

### Logging

**Structured logging** with Python's `logging` module:
```python
import logging
logger = logging.getLogger(__name__)

logger.info("Message received", extra={"user_id": user_id})
logger.error("API call failed", extra={"error": str(e)})
```

---

## Security Best Practices

### Secrets Management

❌ **NEVER commit secrets to repo**
✅ **Always use Google Secret Manager for production**
✅ **Use `.env.local` (gitignored) for local development**
✅ **Rotate tokens periodically**

---

## Resources & Links

**Official Docs:**
- [PyWa Documentation](https://pywa.readthedocs.io)
- [Gemini API Documentation](https://ai.google.dev/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [WhatsApp Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api)

---

**Last Updated:** 2025-01-12
**Cursor Rules Version:** 1.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rfurkan37) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
