## mcp

> This outlines how we build **structured perfection** for [Project Name] with a Python-based API server, extensions, and more.

# Trish's guide to a wonder colaboratie experience with Aye, Hue, Trish, Omni and more

## 📐 Code Style and Structures

This outlines how we build **structured perfection** for [Project Name] with a Python-based API server, extensions, and more.

### 🖊️ **General Guidelines**:
- Keep code **concise and technical** while providing meaningful inline comments.
- Use **functional programming patterns** and avoid unnecessary duplication.
- Python APIs must follow **clean modular design principles** (don’t repeat yourself, even in error handling).
- Use descriptive variable names throughout (e.g., `is_valid_user`, `has_token_expired`).

---

### 🗂️ Repository Structure and Organization

Here’s the shiny, revised structure focusing on Python APIs in `server/`:

```bash
server/
├── src/
│   ├── api/             # RESTful API endpoints (FastAPI preferred)
│   ├── models/          # ORM models (e.g., SQLAlchemy, Pydantic)
│   ├── schemas/         # Input/output validation schemas (e.g., Pydantic)
│   ├── services/        # Business logic and service layer
│   ├── utils/           # Reusable helper functions
│   ├── middleware/      # Custom FastAPI middleware
│   ├── tests/           # Unit and integration tests
│   └── __init__.py      # Make folders Python modules

extension/
├── src/
│   ├── background/
│   ├── content/
│   ├── popup/
│   ├── options/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   ├── lib/
│   ├── types/
│   └── storage/

shared/
├── src/
│   ├── types/           # Shared TypeScript types
│   └── utils/           # Shared utilities (e.g., date formatting)
```

👀 **Important**: Add `.venv` to `.gitignore` (we don’t want Hue accidentally version-controlling the virtual environment)! A sample `.gitignore` for Python:

```gitignore
# Ignore Python virtual environment
.venv/

# Ignore common Python artifacts
*.pyc
__pycache__/
```

---

### 🐍 Setting Up Your Python Environment

To keep things smooth and sandboxed:
1. Use `python -m venv .venv` to create a local virtual environment.
2. Activate it:
   - **Mac/Linux**: `source .venv/bin/activate`
   - **Windows**: `.venv\Scripts\activate`
3. Upgrade `pip` right away to avoid package chaos:
   ```bash
   pip install --upgrade pip
   ```

---

### Python API-Specific Notes

#### 💡 Recommendations:
- Use **FastAPI** for the API framework. It’s fast, modern, and great for typing. (Plus, it writes beautiful OpenAPI documentation automatically! 💅)
- ORM: **SQLAlchemy** or **Tortoise ORM** (pick your favorite).  
- Validation: Use **Pydantic** for data validation and serialization.

#### 🚦 API Folder Breakdown:
Here’s how your `server/src/api/` folder could be structured:

```bash
server/src/api/
├── v1/                  # Versioned API structure
│   ├── users.py         # User endpoints
│   ├── auth.py          # Auth endpoints
│   ├── health.py        # Health checks
│   └── __init__.py      
├── dependencies.py      # Dependency injection helpers
├── errors.py            # Custom API exception handling
├── __init__.py
```

#### 🎯 Sample FastAPI Endpoint:
```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter()

class User(BaseModel):
    username: str
    email: str

@router.post("/users", summary="Create a new user")
async def create_user(user: User):
    if not user.email.endswith("@example.com"):
        raise HTTPException(status_code=400, detail="Invalid email domain")
    # Simulate user saving
    return {"status": "success", "data": user.dict()}
```

---

### 🚀 Server Management Script

Let’s keep day-to-day server tasks hassle-free with a `scripts/manage.sh` script:

```bash
#!/bin/bash

ACTION=${1:-help}

case "$ACTION" in
  "start")
    echo "🚀 Starting the Python server..."
    uvicorn src.api.main:app --reload
    ;;
  "stop")
    echo "🛑 Stopping the Python server (assuming Docker or background process)..."
    ;;
  "test")
    echo "🧪 Running tests..."
    pytest --cov=src
    ;;
  "setup")
    echo "🔧 Setting up the environment..."
    python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
    ;;
  "help"|"")
    echo "🎛️ Usage: scripts/manage.sh {start|stop|test|setup}"
    ;;
  *)
    echo "❌ Unknown action: $ACTION"
    exit 1
    ;;
esac
```

Run this script for setup, testing, and more! Hue, enjoy the added flair in your terminal ("spicy life," as Trisha would say 🌶️).

---

### 🧪 Testing and Validation

- Utilize **pytest** as your test runner (`pip install pytest pytest-cov`).  
- Write **unit tests** for utils and services, and **integration tests** for APIs.  
- Example:

```python
from fastapi.testclient import TestClient
from src.api.main import app

client = TestClient(app)

def test_health_check():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

---

### 🛡️ Security for APIs

Hue, remember: **Security is paramount**. Here's your checklist:
1. **Validate and sanitize all inputs** (trust is earned, not blindly given 😜).  
2. Follow **OAuth2** or **token-based authentication** practices for APIs.
3. Ensure proper **CORS policies** and restrict origins.
4. Implement **rate limiting** to prevent abuse.
5. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/8bit-wraith) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
