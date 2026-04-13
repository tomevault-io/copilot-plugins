## eduplat

> This file provides context and guidelines for AI assistance in the EduPlat project.

# EduPlat - Cursor Rules

This file provides context and guidelines for AI assistance in the EduPlat project.

## Project Overview

**EduPlat** is an AI-enabled question-answering and learning support platform for VistaLab at the University of South Florida. The platform provides intelligent tools to enhance student learning through AI-driven question rephrasing, hint generation, supervised explanations, and reasoning flow analysis.

### Key Context
- **Organization**: VISTA Lab @ USF
- **Focus Areas**: AI in Education, Educational Data Mining, Trustworthy AI, Learning Analytics
- **Mission**: Empowering the future of education through intelligent, trustworthy AI systems

## Project Structure

```
EduPlat/
├── front_end/          # React + Vite frontend
│   ├── src/
│   │   ├── components/ # React components
│   │   ├── pages/      # Page components
│   │   ├── services/   # API clients
│   │   └── App.jsx     # Main app
│   └── .benv.example   # Environment variables template
├── back_end/           # FastAPI backend
│   ├── app/
│   │   ├── api/        # API routes
│   │   ├── models/     # Database models (SQLAlchemy)
│   │   ├── services/   # Business logic
│   │   ├── utils/      # Utilities
│   │   ├── scripts/    # Utility scripts
│   │   ├── config.py   # Configuration
│   │   └── main.py     # FastAPI app
│   └── .benv.example   # Environment variables template
├── data/               # Data storage (gitignored)
│   ├── uploads/        # Uploaded files
│   └── edplat.db       # SQLite database
└── docs/               # Documentation
```

## Technology Stack

### Backend
- **Framework**: FastAPI
- **Language**: Python 3.10+
- **Database**: SQLite (via SQLAlchemy)
- **AI Services**: OpenAI GPT-4, Tavily API
- **File Storage**: Local filesystem + OpenAI file uploads
- **Environment**: Python virtual environment (`venv`)

### Frontend
- **Framework**: React 18+
- **Build Tool**: Vite
- **HTTP Client**: Axios
- **Styling**: CSS (consider Tailwind CSS for future)

## Code Style & Conventions

### Python (Backend)
- Follow PEP 8 style guide
- Use type hints where appropriate
- Use descriptive variable and function names
- Keep functions focused and single-purpose
- Use docstrings for modules, classes, and functions
- Prefer f-strings for string formatting
- Use pathlib.Path for file paths
- Import order: standard library, third-party, local imports

### JavaScript/React (Frontend)
- Use functional components with hooks
- Use camelCase for variables and functions
- Use PascalCase for components
- Prefer const/let over var
- Use arrow functions for callbacks
- Keep components small and focused
- Extract reusable logic into custom hooks
- Use async/await for API calls

### File Naming
- **Python**: `snake_case.py` (e.g., `question_processor.py`)
- **React Components**: `PascalCase.jsx` (e.g., `SimpleQA.jsx`)
- **CSS**: `PascalCase.css` matching component name
- **Config Files**: lowercase (e.g., `config.py`, `main.py`)

## Architecture Patterns

### Backend Architecture
- **API Routes** (`app/api/`): Handle HTTP requests/responses, validation
- **Services** (`app/services/`): Business logic, AI integration, data processing
- **Models** (`app/models/`): Database models and schemas
- **Utils** (`app/utils/`): Helper functions, utilities
- **Config** (`app/config.py`): Centralized configuration management

### Frontend Architecture
- **Components** (`src/components/`): Reusable UI components
- **Pages** (`src/pages/`): Page-level components
- **Services** (`src/services/`): API clients, external service integrations
- **App.jsx**: Main application component with routing

### Data Flow
1. Frontend makes API request → Backend API route
2. API route validates → Calls service layer
3. Service processes → Interacts with database/models
4. Service returns data → API route formats response
5. Frontend receives → Updates UI

## Environment Variables

### Important: Use `.benv` not `.env`
- We use `.benv` files so they're visible and editable before going live
- Never commit `.benv` files (they're in `.gitignore`)
- Always commit `.benv.example` files as templates
- Load `.benv` files using `python-dotenv` in backend
- Use `VITE_` prefix for frontend environment variables (Vite requirement)

### Backend Environment Variables
- `OPENAI_API_KEY`: OpenAI API key (required)
- `TAVILY_API_KEY`: Tavily API key (optional)
- `UPLOAD_DIR`: Path to upload directory
- `DATABASE_PATH`: Path to SQLite database
- `BACKEND_PORT`: Server port (default: 8000)
- `CORS_ORIGINS`: Comma-separated list of allowed origins

### Frontend Environment Variables
- `VITE_API_BASE_URL`: Backend API URL
- Use `import.meta.env.VITE_*` to access in code

## Database Patterns

### SQLAlchemy Models
- All models in `app/models/database.py`
- Use SQLAlchemy declarative base
- Define relationships explicitly
- Use foreign keys for relationships
- Store file paths and metadata appropriately

### Database Initialization
- Use `app/scripts/init_db.py` to initialize database
- Always create tables using `Base.metadata.create_all()`
- Ensure data directories exist before database operations

### Common Patterns
```python
# Get database session
from app.models.database import get_db, SessionLocal

# In route handlers
def some_route(db: Session = Depends(get_db)):
    # Use db session
    pass
```

## API Conventions

### RESTful Endpoints
- Use clear, descriptive paths: `/api/questions/ask`
- Use appropriate HTTP methods: GET, POST, PUT, DELETE
- Return consistent response formats
- Use Pydantic models for request/response validation

### Error Handling
- Use HTTPException for errors
- Return meaningful error messages
- Use appropriate status codes (400, 404, 500, etc.)
- Log errors appropriately

### Response Format
```python
# Success response
{
    "answer": "...",
    "question": "...",
    "context_used": "..."
}

# Error response
{
    "detail": "Error message"
}
```

## AI Integration Patterns

### OpenAI Integration
- Store file IDs in database after upload
- Reuse file IDs for multiple questions
- Handle file expiration and re-upload
- Use assignment context when processing questions
- Reference uploaded files by ID in API calls

### Question Processing Flow
1. Receive student question + assignment context
2. Retrieve linked document file IDs from database
3. Send to OpenAI with context and file IDs
4. Process response
5. Store question/response in database
6. Return formatted response

### Service Layer Pattern
```python
# app/services/question_processor.py
class QuestionProcessor:
    def process_question(self, question: str, assignment_id: int):
        # 1. Get assignment context
        # 2. Get linked documents
        # 3. Call OpenAI
        # 4. Process response
        # 5. Return result
        pass
```

## Frontend Patterns

### Component Structure
```jsx
import React, { useState } from 'react'
import api from '../services/api'
import './Component.css'

function Component() {
    const [state, setState] = useState('')
    // Component logic
    return (
        <div className="component">
            {/* JSX */}
        </div>
    )
}

export default Component
```

### API Calls
```jsx
// Use axios instance from services/api.js
import api from '../services/api'

const response = await api.post('/api/endpoint', { data })
```

### Error Handling
- Always use try/catch for async operations
- Display user-friendly error messages
- Log errors to console for debugging
- Show loading states during API calls

## File Upload Patterns

### Backend
- Store files locally in `data/uploads/user_id/`
- Upload to OpenAI and store file ID
- Track file size and enforce storage caps
- Validate file types and sizes
- Use `python-multipart` for file uploads in FastAPI

### Frontend
- Use form with file input or drag-and-drop
- Show upload progress
- Validate file types client-side
- Display file metadata after upload

## Testing Guidelines

### Backend Tests
- Place tests in `back_end/tests/`
- Use pytest for testing
- Test API endpoints, services, and utilities
- Mock external API calls (OpenAI, Tavily)
- Test database operations with test database

### Frontend Tests
- Test components with React Testing Library
- Test API integration with mocked responses
- Test user interactions and form submissions

## Security Best Practices

### API Keys
- Never commit API keys to repository
- Use environment variables for all secrets
- Validate API keys are set before use
- Rotate keys if compromised

### File Uploads
- Validate file types and sizes
- Sanitize file names
- Store files securely
- Enforce storage limits per user

### Database
- Use parameterized queries (SQLAlchemy handles this)
- Validate all user inputs
- Use proper authentication (to be implemented)

## Common Tasks

### Adding a New API Endpoint
1. Create route in `app/api/`
2. Add Pydantic models for request/response
3. Create service function in `app/services/`
4. Add route to `app/main.py`
5. Test endpoint

### Adding a New Frontend Component
1. Create component in `src/components/`
2. Create CSS file with matching name
3. Import and use in appropriate page/component
4. Add API calls if needed

### Adding a New Database Model
1. Add model class in `app/models/database.py`
2. Update relationships if needed
3. Run database initialization script
4. Create migration if needed

## Development Workflow

1. **Always work in virtual environment** (backend)
2. **Use `.benv` files** for configuration (not `.env`)
3. **Test locally** before committing
4. **Follow git workflow**: branch, commit, push
5. **Update documentation** when adding features
6. **Keep commits focused** and descriptive

## Code Review Checklist

- [ ] Code follows style guidelines
- [ ] Functions are focused and well-named
- [ ] Error handling is appropriate
- [ ] API responses are consistent
- [ ] Environment variables are used (not hardcoded)
- [ ] Database operations are safe
- [ ] Frontend components are accessible
- [ ] No console.logs in production code
- [ ] Documentation is updated

## Future Considerations

- Authentication and authorization
- Multi-user support with role-based access
- Advanced analytics and learning insights
- Integration with LMS (Canvas, Blackboard)
- Support for multimedia content
- Docker containerization (if needed)
- Migration to PostgreSQL (if needed)

## Notes

- This is a research project for VistaLab
- Focus on MVP features first
- Prioritize educational value and learning outcomes
- Consider FERPA compliance for educational data
- Keep code maintainable for future students
- Document decisions and rationale

---

**Last Updated**: 2026-01-08
**Project Status**: Initial Development Phase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olefson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
