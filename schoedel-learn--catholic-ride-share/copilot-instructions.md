## catholic-ride-share

> This is a community-driven ride-sharing application connecting Catholics who need transportation to Mass, Confession, prayer events, and church social functions with volunteer drivers.

# Catholic Ride Share - Cursor AI Rules

## Project Overview
This is a community-driven ride-sharing application connecting Catholics who need transportation to Mass, Confession, prayer events, and church social functions with volunteer drivers.

## Tech Stack
- **Backend**: Python 3.11+ with FastAPI
- **Database**: PostgreSQL with PostGIS (geospatial)
- **Cache**: Redis
- **AI/ML**: OpenAI/Anthropic for chatbot, scikit-learn for matching
- **Authentication**: JWT with OAuth2

## Code Style Guidelines

### Python
- Use Python 3.11+ features and type hints
- Follow PEP 8 style guide
- Format with Black (line length: 100)
- Sort imports with isort
- Use Pydantic for data validation
- Prefer async/await for I/O operations
- Write docstrings for all public functions/classes

### Example Function Style
```python
async def get_nearby_drivers(
    latitude: float,
    longitude: float,
    radius_miles: int = 10,
    db: Session = Depends(get_db)
) -> List[DriverProfile]:
    """Get available drivers within specified radius.

    Args:
        latitude: Pickup location latitude
        longitude: Pickup location longitude
        radius_miles: Search radius in miles (default: 10)
        db: Database session

    Returns:
        List of available driver profiles
    """
    # Implementation
```

### Database
- Use SQLAlchemy ORM for all database operations
- Always use parameterized queries (ORM handles this)
- Use Alembic for schema migrations
- PostGIS for geospatial queries
- Index frequently queried columns

### API Endpoints
- Use RESTful conventions
- Return appropriate HTTP status codes
- Use Pydantic schemas for request/response validation
- Include proper error handling
- Document with docstrings (appears in OpenAPI)

### Security
- NEVER commit secrets or API keys
- Always use environment variables for sensitive data
- Hash passwords with bcrypt (via passlib)
- Validate and sanitize all user inputs
- Use JWT tokens with expiration
- Implement rate limiting on authentication endpoints
- Be aware of OWASP Top 10 vulnerabilities

### Testing
- Write tests for all new features
- Use pytest for testing
- Aim for >80% code coverage
- Test both success and error cases
- Use fixtures for common test data

## Project Structure
```
backend/
├── app/
│   ├── api/endpoints/    # Route handlers
│   ├── models/           # SQLAlchemy models
│   ├── schemas/          # Pydantic schemas
│   ├── services/         # Business logic
│   ├── ai/              # AI/ML services
│   ├── core/            # Config, security
│   └── db/              # Database session
```

## Common Tasks

### Adding a New API Endpoint
1. Create Pydantic schemas in `app/schemas/`
2. Add route handler in `app/api/endpoints/`
3. Include router in `app/main.py`
4. Write tests in `tests/`

### Adding a Database Model
1. Create model in `app/models/`
2. Import in `app/models/__init__.py`
3. Create migration: `alembic revision --autogenerate -m "description"`
4. Review and apply: `alembic upgrade head`

### Working with Location Data
- Use PostGIS Geography type for locations
- Store coordinates as POINT(longitude, latitude)
- Use ST_DWithin for radius queries
- Distances in meters (convert from miles: miles * 1609.34)

## AI/ML Integration
- Use OpenAI or Anthropic API for chatbot
- Implement RAG for parish/Mass information
- Store embeddings in vector database (Chroma)
- Cache AI responses when possible

## Business Logic
- No payment for rides (donations only via Stripe)
- 10-mile default radius for driver matching
- Driver verification required
- Parish affiliation optional but encouraged
- Privacy: blur locations until ride accepted

## Development Workflow
1. Create feature branch from `main`
2. Implement feature with tests
3. Run linting and formatting
4. Ensure tests pass
5. Create pull request with description

## Commands
- Start dev: `uvicorn app.main:app --reload`
- Run tests: `pytest`
- Format: `black . && isort .`
- Migrate: `alembic upgrade head`

## Common Patterns

### Dependency Injection
```python
from app.api.deps.auth import get_current_active_user

@router.get("/me")
async def get_profile(
    current_user: User = Depends(get_current_active_user),
    db: Session = Depends(get_db)
):
    return current_user
```

### Error Handling
```python
from fastapi import HTTPException, status

if not resource:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Resource not found"
    )
```

### Geospatial Query Example
```python
from geoalchemy2 import func
from app.core.config import settings

radius_meters = settings.MAX_DRIVER_DISTANCE_MILES * 1609.34
point = func.ST_GeogFromText(f'POINT({longitude} {latitude})')

nearby_drivers = db.query(DriverProfile).join(User).filter(
    func.ST_DWithin(
        User.location,
        point,
        radius_meters
    ),
    DriverProfile.is_available == True
).all()
```

## Important Notes
- This serves the Catholic community - maintain appropriate and respectful code
- Privacy is critical - handle location data carefully
- Accessibility matters - consider users with disabilities
- Multi-language support is planned - structure for i18n

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/schoedel-learn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
