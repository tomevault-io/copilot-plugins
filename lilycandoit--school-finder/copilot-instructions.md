## school-finder

> Enforce best practices for FastAPI MVC architecture with TailwindCSS, SQLite ORM, Fly.io deployment, and GitHub version control.


# FastAPI MVC Development Guide

## 🧱 Architecture & Project Structure

**Description**: Standard MVC organization for FastAPI projects with clear separation of concerns.

### Project Structure
```
project/
├── app/
│   ├── __init__.py
│   ├── models/          # Database models (SQLAlchemy/SQLModel)
│   ├── controllers/     # Business logic layer
│   ├── routes/          # API route definitions
│   ├── templates/       # Jinja2 templates with TailwindCSS
│   ├── utils/           # Utility functions
│   └── database.py      # Database configuration
├── tests/               # Test files
├── migrations/          # Alembic migrations
├── main.py             # FastAPI application entry point
├── requirements.txt    # Dependencies
├── fly.toml            # Fly.io deployment config
└── README.md           # Project documentation
```

### File Responsibilities
- **Models** (`app/models/`): SQLAlchemy/SQLModel database models
- **Controllers** (`app/controllers/`): Business logic and data access operations
- **Routes** (`app/routes/`): HTTP request/response handling and route definitions
- **Templates** (`app/templates/`): Jinja2 templates with TailwindCSS styling
- **Utils** (`app/utils/`): Shared utility functions and helpers

---

## ⚙️ FastAPI Best Practices

**Description**: Core FastAPI patterns for application setup, routing, and dependency injection.

### Application Setup
Use lifespan context managers for startup/shutdown logic.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Manage application lifespan."""
    await init_db()  # Startup
    yield
    # Shutdown

app = FastAPI(
    title="API Title",
    description="API Description", 
    version="1.0.0",
    lifespan=lifespan
)

app.mount("/static", StaticFiles(directory="app/static"), name="static")
templates = Jinja2Templates(directory="app/templates")
```

### Route Organization
Organize routes by feature using `APIRouter` with OpenAPI tags.

```python
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1", tags=["todos"])

@router.post("/todos", response_model=TodoResponse, tags=["todos"])
async def create_todo_endpoint(
    todo: TodoCreate,
    db: AsyncSession = Depends(get_db)
) -> TodoResponse:
    """Create a new todo."""
    return await create_todo(db, todo)
```

### Dependency Injection
- Use `Depends()` for database connections, authentication, and services
- Create reusable dependencies for common operations
- Use async dependencies for I/O operations

---

## 🎨 Frontend Best Practices

**Description**: TailwindCSS + Vanilla JavaScript frontend patterns with mobile-first responsive design.

### Template Structure
Use Jinja2 templates with FastAPI for server-side rendering.

```python
from fastapi import Request
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse

templates = Jinja2Templates(directory="app/templates")

@app.get("/", response_class=HTMLResponse)
async def read_root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```

### Base Template
Standard base template with TailwindCSS CDN.

```html
<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}{{ title or "FastAPI App" }}{% endblock %}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    {% block head %}{% endblock %}
</head>
<body class="h-full bg-gray-50">
    <div class="min-h-full">
        {% block content %}{% endblock %}
    </div>
    {% block scripts %}{% endblock %}
</body>
</html>
```

### HTML Best Practices
- Use semantic HTML5 elements (`<header>`, `<main>`, `<section>`, etc.)
- Include proper meta tags for SEO and responsive design
- Use `lang` attribute on `<html>` element
- Implement proper heading hierarchy (h1 → h2 → h3)
- Use descriptive alt text for images
- Include ARIA labels for accessibility

### TailwindCSS Guidelines
- Use utility-first approach with consistent spacing scale
- Leverage responsive prefixes (`sm:`, `md:`, `lg:`, `xl:`)
- Use CSS Grid and Flexbox utilities for layouts
- Use state variants (`hover:`, `focus:`, `active:`, `disabled:`)
- Optimize for mobile-first responsive design

### Mobile-First Design
**Core Principles**: Design for mobile first, then enhance for larger screens.

**Responsive Breakpoints**:
- Base: 320px+ (mobile)
- `sm:`: 640px+ (tablet)
- `md:`: 768px+ (small desktop)
- `lg:`: 1024px+ (desktop)
- `xl:`: 1280px+ (large desktop)

**Mobile-First Patterns**:
```html
<!-- Responsive grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
  <!-- Content adapts from 1 column on mobile to 4 on xl screens -->
</div>

<!-- Responsive typography -->
<h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
  Responsive Heading
</h1>

<!-- Responsive spacing -->
<div class="p-4 sm:p-6 md:p-8 lg:p-12">
  <!-- Padding increases with screen size -->
</div>
```

**Mobile Considerations**:
- Touch targets: Minimum 44px (11 in Tailwind)
- Viewport meta tag: Always include
- Test on actual devices, not just browser dev tools
- Optimize for performance (smaller files, faster loading)

### Component Patterns
Standard component patterns for cards, forms, and navigation.

```html
<!-- Card Component -->
<div class="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
    <h3 class="text-lg font-semibold text-gray-900 mb-2">{{ card.title }}</h3>
    <p class="text-gray-600 mb-4">{{ card.description }}</p>
    <button class="bg-primary text-white px-4 py-2 rounded-md hover:bg-blue-700">
        {{ card.button_text }}
    </button>
</div>

<!-- Form Component -->
<form class="space-y-4">
    <div>
        <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
            Email Address
        </label>
        <input 
            type="email" 
            id="email" 
            name="email"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-primary"
            required
        >
    </div>
</form>
```

### Vanilla JavaScript Best Practices
- Use modern ES6+ features (arrow functions, destructuring, async/await)
- Implement proper error handling with try/catch blocks
- Use event delegation for dynamic content
- Follow functional programming principles
- Use fetch API for HTTP requests with proper error handling

### Frontend Security
- **Input Sanitization**: Jinja2 auto-escapes by default
- **CSP**: Implement strict Content Security Policy headers
- **CSRF Protection**: Use CSRF tokens for state-changing operations
- **HTTPS**: Enforce HTTPS in production
- **Secure Cookies**: Use HttpOnly, Secure, and SameSite attributes
- **XSS Prevention**: Never use `innerHTML` with user input

### Performance Optimization
- Minimize HTTP requests
- Use Tailwind's purge feature to remove unused styles
- Implement lazy loading for images
- Use modern image formats (WebP, AVIF)
- Minimize JavaScript bundle size
- Use browser caching effectively

### Accessibility (a11y)
- Use semantic HTML elements
- Implement proper focus management
- Include ARIA labels and roles
- Ensure sufficient color contrast
- Provide keyboard navigation support

---

## 🎯 Coding Standards & Best Practices

**Description**: Python/FastAPI coding conventions and best practices.

### Key Principles
- Write concise, technical code with accurate Python examples
- Use functional, declarative programming; avoid classes where possible
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., `is_active`, `has_permission`)
- Use lowercase with underscores for directories and files
- Use the Receive an Object, Return an Object (RORO) pattern

### Python/FastAPI Guidelines
- Use `def` for pure functions and `async def` for asynchronous operations
- Use type hints for all function signatures
- Prefer Pydantic models over raw dictionaries for input validation
- Use declarative route definitions with clear return type annotations
- Prefer lifespan context managers over `@app.on_event()`

### Error Handling
- Handle errors and edge cases at the beginning of functions
- Use early returns for error conditions
- Place the happy path last in the function
- Use guard clauses to handle preconditions early
- Implement proper error logging and user-friendly error messages

### Performance Optimization
- Minimize blocking I/O operations
- Use async operations for all database calls and external API requests
- Implement caching for static and frequently accessed data
- Optimize data serialization with Pydantic
- Use lazy loading for large datasets

---

## 🗃️ Database (SQLite + ORM)

**Description**: SQLAlchemy/SQLModel setup and CRUD operations with async SQLite.

### Database Setup
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from sqlmodel import SQLModel

DATABASE_URL = "sqlite+aiosqlite:///./database.db"
engine = create_async_engine(DATABASE_URL, echo=True)
async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """Get database session."""
    async with async_session() as session:
        yield session

async def init_db() -> None:
    """Initialize database and create tables."""
    async with engine.begin() as conn:
        await conn.run_sync(SQLModel.metadata.create_all)
```

### Database Models
```python
from sqlmodel import SQLModel, Field
from typing import Optional

class Todo(SQLModel, table=True):
    """Database model for todos."""
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    completed: bool = Field(default=False)
```

### CRUD Operations
```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlmodel import select

async def create_todo(db: AsyncSession, todo: TodoCreate) -> TodoResponse:
    """Create a new todo."""
    db_todo = Todo(title=todo.title, completed=False)
    db.add(db_todo)
    await db.commit()
    await db.refresh(db_todo)
    return TodoResponse.from_orm(db_todo)

async def get_todo(db: AsyncSession, todo_id: int) -> Optional[TodoResponse]:
    """Get a specific todo by ID."""
    result = await db.execute(select(Todo).where(Todo.id == todo_id))
    todo = result.scalar_one_or_none()
    return TodoResponse.from_orm(todo) if todo else None
```

### Migration Management
- Use Alembic for database migrations
- Store migration files in `migrations/` directory
- Always test migrations on staging before production

---

## Error Handling

**Description**: HTTP status codes and exception patterns for FastAPI.

### HTTP Status Codes
- **200**: Success (GET, PUT)
- **201**: Created (POST)
- **204**: No Content (DELETE)
- **400**: Bad Request (validation errors)
- **404**: Not Found (resource not found)
- **422**: Unprocessable Entity (Pydantic validation errors)
- **500**: Internal Server Error (unexpected errors)

### Exception Patterns
```python
from fastapi import HTTPException

# Resource not found
if not todo:
    raise HTTPException(status_code=404, detail="Todo not found")

# Database error handling
try:
    # Database operation
    await db.commit()
except Exception as e:
    await db.rollback()
    raise HTTPException(status_code=500, detail="Database error")
```

---

## Type Safety and Validation

**Description**: Type hints and Pydantic validation patterns.

### Type Hints
- Always use type hints for function parameters and return types
- Use `List[Model]` for collections
- Use `Optional[Model]` for nullable returns
- Use `AsyncGenerator` for database connections

### Pydantic Validation
- FastAPI automatically validates request bodies using Pydantic models
- Returns 422 status code for validation errors
- Provides detailed error messages for invalid fields

---

## 🔒 Security Best Practices

**Description**: Comprehensive security guidelines for FastAPI applications.

### Dependency Security
- Use `pip-audit` or `safety` to scan for vulnerabilities
- Pin exact versions in `requirements.txt` for production (`==`)
- Regularly update dependencies
- Set up automated alerts (GitHub Dependabot, Snyk)

### SQL Injection Prevention
**Always use parameterized queries or ORM** - Never concatenate user input.

```python
# ✅ CORRECT: Use ORM
result = await db.execute(select(User).where(User.id == user_id))

# ✅ CORRECT: Parameterized queries
result = await db.execute(text("SELECT * FROM users WHERE id = :user_id"), {"user_id": user_id})

# ❌ WRONG: Never do this
# result = await db.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

### XSS Prevention
- Jinja2 auto-escapes by default
- Never use `innerHTML` with user input
- Use `textContent` or sanitize with DOMPurify
- Implement strict CSP headers

### CSRF Protection
Use CSRF tokens for all state-changing operations.

```python
from fastapi_csrf_protect import CsrfProtect

@app.post("/create")
async def create_item(request: Request, csrf_protect: CsrfProtect = Depends()):
    await csrf_protect.validate_csrf(request)
    # Process request...
```

### Input Validation
Use Pydantic models for all input validation.

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    password: str = Field(..., min_length=8)
```

### Authentication & Authorization
- Use secure session management with HttpOnly cookies
- Hash passwords with bcrypt (never store plain text)
- Use cryptographically secure tokens
- Implement proper authorization checks

### Password Security
```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

### Secure Session Management
```python
import secrets
from datetime import datetime, timedelta

async def create_session(user_id: int, db: AsyncSession):
    token = secrets.token_urlsafe(32)  # Cryptographically secure
    expires_at = datetime.utcnow() + timedelta(days=1)
    # ... create session ...

# Set secure cookie
response.set_cookie(
    key="session_token",
    value=session.token,
    httponly=True,  # Prevents JavaScript access
    secure=True,     # HTTPS only
    samesite="lax", # CSRF protection
    max_age=86400
)
```

### API Security
- Implement rate limiting to prevent abuse
- Use rate limiting libraries like `slowapi`
- Restrict CORS to specific origins (never use `["*"]` in production)

### File Upload Security
- Validate file extensions and sizes
- Verify file type using magic bytes (not just extension)
- Save files with random filenames outside web root
- Use secure file paths

### Secrets Management
- **Never hardcode secrets** in code
- Use environment variables for all secrets
- Use `secrets` module for cryptographic operations
- Never use predictable or weak secrets

### Security Headers
Add security headers middleware for protection.

```python
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
    return response
```

### CORS Configuration
Restrict CORS to specific origins in production.

```python
from fastapi.middleware.cors import CORSMiddleware

ALLOWED_ORIGINS = ["https://yourdomain.com"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=ALLOWED_ORIGINS,  # Never use ["*"] in production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
)
```

### Security Checklist
**Pre-Deployment**:
- [ ] All dependencies are up-to-date and vulnerability-free
- [ ] All user inputs validated using Pydantic models
- [ ] SQL queries use parameterized queries or ORM
- [ ] Passwords hashed using bcrypt
- [ ] Session tokens are cryptographically secure and HttpOnly
- [ ] CSRF protection enabled for state-changing operations
- [ ] File uploads validated (type, size, content)
- [ ] Error messages don't expose sensitive information
- [ ] Security headers configured
- [ ] CORS restricted to specific origins
- [ ] Rate limiting implemented
- [ ] Secrets stored in environment variables
- [ ] HTTPS enforced in production

---

## Development Guidelines

**Description**: Code organization and testing considerations.

### Code Organization
- Keep functions focused and single-purpose
- Use descriptive variable names
- Prefer functional programming over classes
- Use early returns for error conditions
- Place the happy path last in functions

### Error Handling Strategy
1. **Early Validation**: Validate input as early as possible
2. **Clear Messages**: Provide clear, actionable error messages
3. **Consistent Format**: Use consistent error response format
4. **Graceful Degradation**: Handle errors gracefully
5. **Security**: Don't expose sensitive information

### Testing Considerations
- Each layer can be tested independently
- Use dependency injection for testability
- Mock database connections in unit tests
- Test error conditions and edge cases

---

## Dependencies

**Description**: Required and optional packages for FastAPI projects.

### Required Packages
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
aiosqlite==0.19.0  # or asyncpg, aiomysql for production
```

### Optional Packages
```
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.2  # for testing
```

### Security Tools (Recommended)
```
pip-audit>=2.6.0  # Dependency vulnerability scanning
safety>=2.3.0     # Alternative vulnerability scanner
bandit>=1.7.5     # Security linting for Python code
```

---

## 🚀 Advanced Architecture & Scalability

**Description**: Patterns for building scalable FastAPI applications.

### Microservices Principles
- Design services to be stateless
- Use external storage and caches (Redis) for state
- Implement API gateways and reverse proxies
- Use circuit breakers and retries
- Use asynchronous workers (Celery, RQ) for background tasks

### Performance Optimization
- Leverage FastAPI's async capabilities
- Use caching layers (Redis, Memcached)
- Optimize for high throughput and low latency
- Apply load balancing and service mesh technologies

### Monitoring and Logging
- Use Prometheus and Grafana for monitoring
- Implement structured logging
- Integrate with centralized logging systems (ELK Stack, CloudWatch)

---

## 🚀 Fly.io Deployment & Volume Persistence

**Description**: Fly.io deployment configuration and volume management for persistent data.

### Critical: Always Use Volumes for Persistent Data
**Without volumes, all data will be lost on deployment or machine restart!**

Configure volumes for:
- Database files (`.db` files)
- Log files
- Uploaded files
- Session data
- Any other persistent data

### Fly.toml Volume Configuration
```toml
[app]
app = "your-app-name"
primary_region = "lax"

[[mounts]]
source = "your_app_data"
destination = "/data"
```

### Database Configuration for Fly.io
```python
import os
from pathlib import Path

# Determine database path
if os.path.exists("/data"):
    # Production on Fly.io - use volume
    DB_PATH = Path("/data/vibecamp.db")
else:
    # Development - use local file
    DB_PATH = Path("./vibecamp.db")

DB_PATH.parent.mkdir(parents=True, exist_ok=True)
DATABASE_URL = f"sqlite+aiosqlite:///{DB_PATH}"
```

### Volume Management Commands
```bash
# Create volume
fly volumes create your_app_data --size 10 --region lax

# List volumes
fly volumes list

# Extend volume size
fly volumes extend your_app_data --size 20

# Create snapshot
fly volumes snapshot your_app_data
```

### Database Backup Scripts
Create two scripts in `scripts/` directory:

**1. Backup Script** (`scripts/backup_db.py`):
- Downloads database from Fly.io production volume
- Creates timestamped backup files
- Verifies download success

**2. Push Script** (`scripts/push_db.py`):
- Uploads local database file to Fly.io production volume
- Includes safety confirmation prompt
- Verifies upload success

**Usage**:
```bash
# Backup production database
python3 scripts/backup_db.py

# Push local database to production
python3 scripts/push_db.py --yes
```

### Volume Best Practices
- **Small apps**: Start with 10GB
- **Medium apps**: 20-50GB
- **Large apps**: 100GB+ as needed
- Monitor usage regularly
- Always backup before pushing changes

### Pre-Deployment Checklist
- [ ] Volume created and attached
- [ ] Database path configured to `/data/` directory
- [ ] Log directory configured to `/data/logs/`
- [ ] Upload directory configured to `/data/uploads/`
- [ ] Backup strategy implemented
- [ ] Volume persistence tested

---

## 🎨 Vibecamp Design System

**Description**: Consistent branding and design system for Vibecamp POC applications.

### Brand Identity
- **Brand Name**: "vibed coded" (lowercase, informal)
- **Tagline**: "A Vibecamp Creation" (footer attribution)
- **Logo**: Use `vc-logo.svg` from `/static/img/` directory
- **Color Philosophy**: Pure black and white aesthetic

### Color Palette
```css
--primary: #000000;        /* Pure black */
--secondary: #6B7280;      /* Gray-500 */
--bg-primary: #ffffff;     /* Pure white */
--text-primary: #000000;   /* Black text */
--text-secondary: #6B7280;  /* Gray-500 */
```

### Typography
- **Headings**: 'Kalam', cursive (handwritten style)
- **Body**: 'Inter', system-ui, sans-serif
- **Font Import**: Google Fonts (Kalam, Inter)

### TailwindCSS Configuration
```javascript
tailwind.config = {
  theme: {
    extend: {
      colors: {
        primary: '#000000',
        secondary: '#6B7280'
      },
      fontFamily: {
        'sans': ['Inter', 'system-ui', 'sans-serif'],
        'hand': ['Kalam', 'cursive', 'system-ui']
      }
    }
  }
}
```

### Component Patterns
**Navigation Bar**:
```html
<nav class="bg-white border-b-2 border-black">
    <h1 class="text-2xl font-hand font-bold text-black flex items-center">
        <img src="/static/img/vc-logo.svg" alt="Logo" class="w-8 h-8 mr-2">
        [App Name]
    </h1>
</nav>
```

**Form Elements**:
```html
<div class="bg-white border-2 border-black rounded-lg p-4 sm:p-6">
    <label class="block text-sm font-medium text-black mb-2 font-hand">
        Field Label
    </label>
    <input 
        type="text" 
        class="w-full px-4 py-3 border-2 border-black rounded-lg focus:ring-2 focus:ring-black font-sans"
    >
    <button class="w-full bg-black text-white px-6 py-3 rounded-lg font-hand font-bold">
        Submit
    </button>
</div>
```

**Cards**:
```html
<div class="bg-white border-2 border-black rounded-lg shadow-md p-4 sm:p-6">
    <h3 class="text-lg font-hand font-bold text-black">Card Title</h3>
    <p class="text-gray-600 font-sans">Content text</p>
</div>
```

**Footer**:
```html
<footer class="bg-white border-t-2 border-black py-8">
    <div class="text-center">
        <p class="text-sm text-black font-hand">
            A <a href="https://vibecamp.au/" class="font-bold">Vibecamp</a> Creation
        </p>
    </div>
</footer>
```

### Design Principles
1. **Consistency**: Same color palette, typography, and spacing across all apps
2. **Simplicity**: Clean black and white design
3. **Accessibility**: High contrast ratios and readable fonts
4. **Mobile-First**: Design for mobile first, then enhance
5. **Brand Recognition**: Always include Vibecamp logo and attribution
6. **Informal Tone**: Handwritten-style fonts for headings

### Implementation Checklist
- [ ] Add logo to `/static/img/vc-logo.svg`
- [ ] Configure TailwindCSS with brand colors and fonts
- [ ] Import Google Fonts (Kalam, Inter)
- [ ] Apply consistent navigation structure
- [ ] Use black borders (`border-2 border-black`) for containers
- [ ] Apply `font-hand` for headings
- [ ] Apply `font-sans` for body text
- [ ] Include footer attribution
- [ ] Test mobile responsiveness
- [ ] Ensure high contrast accessibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lilycandoit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
