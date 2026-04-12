## kappi1

> Development workflow — how to add features, find code, test changes


# Development Workflow

## Running the Project

```bash
# Development (hot reload)
docker-compose up -d          # MongoDB :27017, Backend :8001, Frontend :3000

# Alternative dev ports
docker-compose -f docker-compose-dev.yml up -d   # Backend :435, Frontend :415

# Production
docker-compose -f docker-compose.prod.yml up -d
```

Backend runs on `uvicorn server:app --reload`, frontend on `craco start`.

## Adding a New Backend Feature

### 1. Define the model (`backend/models/`)

Create or extend a Pydantic model. Follow existing conventions:

```python
from pydantic import BaseModel, Field
from datetime import datetime, timezone
import uuid

class MyEntity(BaseModel):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    name: str
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
```

### 2. Create the API router (`backend/api/api_my_feature.py`)

```python
from fastapi import APIRouter, Depends, HTTPException
from config.config_init import db, logger
from models.auth_models import User
from services.services_auth import get_current_user, require_permission
from utils.db_utils import prepare_for_mongo, parse_from_mongo

router = APIRouter(prefix="/my-feature", tags=["my-feature"])

@router.get("")
async def list_items(current_user: User = Depends(get_current_user)):
    await require_permission(current_user, "my_feature_view")
    docs = await db.my_collection.find({}, {"_id": 0}).to_list(1000)
    return [parse_from_mongo(d) for d in docs]

@router.post("")
async def create_item(body: MyEntityCreate, current_user: User = Depends(get_current_user)):
    await require_permission(current_user, "my_feature_edit")
    doc = prepare_for_mongo(body.model_dump())
    await db.my_collection.insert_one(doc)
    return parse_from_mongo(doc)
```

### 3. Register the router (`backend/api/__init__.py`)

```python
from .api_my_feature import router as my_feature_router
api_router.include_router(my_feature_router)
```

### 4. Add permissions (`backend/config/config_settings.py`)

Add to the `PERMISSIONS` dict and appropriate permission group:

```python
PERMISSIONS = {
    # ...existing...
    "my_feature_view": "Просмотр моей функции",
    "my_feature_edit": "Редактирование моей функции",
}
```

### 5. Add DB indexes (if needed) (`backend/config/config_database.py`)

```python
async def ensure_indexes():
    # ...existing...
    await db.my_collection.create_index("name")
```

## Adding a New Frontend Page

### 1. Create the page component (`frontend/src/pages/MyFeaturePage.jsx`)

```jsx
import { useState, useEffect } from 'react';
import api from '../config/api';
import { useAuth } from '../contexts/AuthContext';

export default function MyFeaturePage() {
  const { hasPermission } = useAuth();
  const [items, setItems] = useState([]);

  useEffect(() => {
    api.get('/api/my-feature').then(r => setItems(r.data));
  }, []);

  return (/* JSX */);
}
```

### 2. Add the route (`frontend/src/App.js`)

```jsx
const MyFeaturePage = lazy(() => import('./pages/MyFeaturePage'));

// Inside ProtectedRoute:
<Route path="/my-feature" element={<MyFeaturePage />} />
```

### 3. Add sidebar link (`frontend/src/components/layouts/MainLayout.jsx`)

Add to the navigation items array with the appropriate permission check.

## Where to Find Code

| Looking for… | Look in… |
|--------------|----------|
| API endpoint handler | `backend/api/api_*.py` — file matches domain name |
| Business logic | `backend/services/services_*.py` |
| Data model / validation | `backend/models/*.py` |
| DB queries | Usually inline in `api/` or `services/` — search for `db.collection_name` |
| Permission definitions | `backend/config/config_settings.py` → `PERMISSIONS` dict |
| Frontend page | `frontend/src/pages/` — file name matches route |
| UI component | `frontend/src/components/ui/` — shadcn/ui Radix-based |
| API client config | `frontend/src/config/api.js` |
| Auth logic (frontend) | `frontend/src/contexts/AuthContext.js` |
| Route definitions | `frontend/src/App.js` |
| Sidebar navigation | `frontend/src/components/layouts/MainLayout.jsx` |

## Testing

### Backend

```bash
cd backend
pytest                       # Run all tests
pytest tests/unit/           # Unit tests only
pytest --cov=. --cov-report=html   # With coverage
```

Test fixtures are in `backend/tests/fixtures/builders.py`. Mocks in `backend/tests/mocks/`.

### Frontend

```bash
cd frontend
yarn test                    # Run tests (watch mode)
yarn test:coverage           # With coverage report
```

MSW handlers are in `frontend/src/__mocks__/handlers.js`.

## Code Style Conventions

### Backend
- Async everywhere — all DB operations use `await`
- Import config from `config.config_init`, not individual config files
- Use `prepare_for_mongo()` before DB insert, `parse_from_mongo()` after read
- Call `log_audit()` for user-facing actions
- Permissions: `require_permission(current_user, "perm_name")` at the start of each endpoint

### Frontend
- Functional components only
- shadcn/ui components from `components/ui/`
- Tailwind CSS for styling, no inline styles
- `api.get/post/put/delete` for HTTP calls (auto-handles auth)
- `useAuth()` for permission checks: `hasPermission("perm_name")`
- Lazy load all pages in `App.js`

## Deployment Checklist

1. Set secure `JWT_SECRET_KEY` and `ENCRYPTION_KEY` in environment
2. Set `CORS_ORIGINS` to actual frontend domain
3. Change default admin password after first login
4. Configure MongoDB authentication
5. Use `docker-compose.prod.yml` or `docker-compose.offline.yml`
6. Verify health: `GET /api/health`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AMnEPka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AMnEPka)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
