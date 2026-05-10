## my-rule

> FastAPI + React + LangChain intelligent customer service system supporting multiple LLM providers (OpenAI, Gemini, DeepSeek, Zhipu, Baichuan, Qwen, Moonshot, Yi).


# 🤖 AI Customer Service System - Cursor Rules

## 📋 Project Overview

FastAPI + React + LangChain intelligent customer service system supporting multiple LLM providers (OpenAI, Gemini, DeepSeek, Zhipu, Baichuan, Qwen, Moonshot, Yi).

**Tech Stack**: Python 3.9+, FastAPI, LangChain, ChromaDB, React 18, TypeScript, Ant Design

---

## 🎯 Core Principles

1. **Keep It Simple** - Write clean, minimal code. Avoid redundancy.
2. **Stability First** - Never break existing functionality.
3. **No Random Dependencies** - DO NOT add packages/files unless absolutely necessary.
4. **Well-Commented** - All code must have clear comments for easy maintenance.

---

## 💻 Backend Standards

### Python Code Pattern
```python
# ✅ GOOD
async def get_chat_history(session_id: str, limit: int = 50) -> List[Dict]:
    """
    Retrieve chat history for a session.
    
    Args:
        session_id: Unique session identifier
        limit: Maximum messages to return
    
    Returns:
        List of chat messages
    """
    logger.info(f"Fetching history: session_id={session_id}")
    return await memory_service.get_history(session_id, limit)

# ❌ BAD
def getChatHistory(sessionId, limit=50):  # camelCase, no async, no comments
    print("getting history")  # Use logger!
    return memory_service.get_history(sessionId, limit)
```

### Exception Handling
```python
try:
    result = await service.process()
    return create_response(data=result, message="Success")
except ValueError as e:
    logger.warning(f"Validation error: {e}")
    raise HTTPException(status_code=400, detail=str(e))
except Exception as e:
    logger.error(f"Error: {e}\n{traceback.format_exc()}")
    raise HTTPException(status_code=500, detail="Internal error")
```

### Dependency Injection
```python
def get_service() -> Service:
    """Get service instance."""
    return Service()

@router.post("/endpoint")
async def handler(request: Request, service: Service = Depends(get_service)):
    """Handle request with injected service."""
    return await service.process(request)
```

---

## 🎨 Frontend Standards

### React Components
```typescript
// ✅ GOOD: Functional component with comments
interface ChatMessageProps {
  message: Message;
  isUser: boolean;
}

/**
 * Display a single chat message.
 * Renders differently for user vs bot.
 */
const ChatMessage: React.FC<ChatMessageProps> = ({ message, isUser }) => {
  return (
    <div className={isUser ? 'user-message' : 'bot-message'}>
      {message.content}
    </div>
  );
};

// ❌ BAD: Class components, no comments
class ChatMessage extends React.Component { }
```

### API Calls
```typescript
/**
 * Send message to chat API.
 * Handles errors with user notifications.
 */
const sendMessage = async (message: string) => {
  try {
    const response = await chatService.sendMessage({
      message,
      session_id: sessionId
    });
    return response.data;
  } catch (error) {
    message.error('Failed to send. Retry.');
    console.error('Send error:', error);
  }
};
```

---

## 🏗️ Project Structure

### Backend
```
backend/
├── services/        # Business logic (chat, memory, knowledge)
├── models/          # Pydantic models + AI model adapter
├── api/             # API routes
├── routes.py        # Main route definitions
├── config.py        # Settings (environment variables)
└── main.py          # Application entry
```

### Frontend
```
frontend/src/
├── components/      # Reusable components (ChatMessage, Header, etc.)
├── pages/           # Page components (ChatPage, SettingsPage)
├── services/        # API service layer
└── types/           # TypeScript types
```

---

## 🔧 Common Tasks

### Adding API Endpoint

**1. Model** (`backend/models/chat_models.py`):
```python
class NewRequest(BaseModel):
    """Request model."""
    field: str
```

**2. Route** (`backend/routes.py`):
```python
@router.post("/endpoint")
async def handler(request: NewRequest):
    """Handle request."""
    return create_response(data={"result": "success"})
```

**3. Frontend** (`frontend/src/services/`):
```typescript
export const callEndpoint = async (data: NewRequest) => {
  return await api.post('/api/endpoint', data);
};
```

### Adding AI Model

**1. Env** (`.env`):
```env
MODEL_API_KEY=sk-xxx
MODEL_API_BASE=https://api.example.com
```

**2. Config** (`config.py`):
```python
model_api_key: str = os.getenv("MODEL_API_KEY", "")
```

**3. Adapter** (`model_adapter.py`):
```python
if provider == "new_model":
    return ChatOpenAI(
        api_key=settings.model_api_key,
        base_url=settings.model_api_base
    )
```

---

## 📊 API Response Format

```json
{
  "success": true,
  "code": 200,
  "message": "Operation successful",
  "data": {},
  "timestamp": "2024-01-01T12:00:00"
}
```

---

## 🐛 Debugging

### Backend
```python
from loguru import logger
logger.debug(f"Debug: {var}")
logger.info(f"Info: {msg}")
logger.error(f"Error: {e}\n{traceback.format_exc()}")
```

### Logs
- Backend: `backend/logs/app.log`
- Frontend: Browser console
- Docker: `docker-compose logs -f`

---

## 🔐 Security

```python
# ✅ GOOD: Environment variables
api_key = os.getenv("OPENAI_API_KEY")

# ❌ BAD: Hardcoded (FORBIDDEN!)
api_key = "sk-xxxxx"
```

**Never commit** `.env` files!

---

## 📝 Pre-Commit Checklist

- [ ] Code follows PEP 8 / ESLint
- [ ] Added comments and docstrings
- [ ] No debug code (`print`, `console.log`)
- [ ] No sensitive data (API keys)
- [ ] Tested functionality
- [ ] Updated docs (if needed)

---

## ⚠️ Critical Rules

### DO NOT
- ❌ Add dependencies without explicit need
- ❌ Create new files unless necessary
- ❌ Use `print()` - use `logger`
- ❌ Write code without comments
- ❌ Break existing functionality
- ❌ Commit `.env` files
- ❌ Use class components (React)

### ALWAYS
- ✅ Comment functions/classes
- ✅ Use async/await for I/O
- ✅ Handle exceptions properly
- ✅ Follow existing patterns
- ✅ Test before committing
- ✅ Keep code simple

---

## 🚀 Quick Commands

```bash
# Backend
cd backend
pip install -r requirements.txt  # or: uv sync
python main.py

# Frontend
cd frontend
npm install && npm start  # Port 1111

# Docker
docker-compose up -d

# API Docs
http://localhost:8000/docs
```

---

**Remember: Simple, Stable, No Random Additions!** 🚀

---
> Source: [EmberRavager/AICustomerService](https://github.com/EmberRavager/AICustomerService) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
