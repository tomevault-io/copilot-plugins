## doktorabc-redesign

> ❌ Never use bare except clauses:

# Python Core Architecture Patterns

## Error Handling

❌ Never use bare except clauses:
```python
try:
    user = db.users.get(user_id)
except:
    return None
```

✅ Always use specific exception handling:
```python
from typing import Optional
from app.exceptions import UserNotFoundError, DatabaseError

@handle_service_errors
async def get_user(user_id: str) -> Optional[User]:
    try:
        return await db.users.get(user_id)
    except UserNotFoundError:
        log.warning(f"User {user_id} not found")
        return None
    except DatabaseError as e:
        log.error(f"Database error fetching user {user_id}: {e}")
        raise
```

## Service Layer Pattern

❌ Never put business logic in route handlers:
```python
@app.route('/users/<user_id>')
def get_user(user_id: str):
    user = db.users.get(user_id)
    if not user:
        return {'error': 'Not found'}, 404
    return {'name': user.name, 'email': user.email}
```

✅ Always separate routes from business logic:
```python
# routes.py
@app.route('/users/<user_id>')
def get_user_route(user_id: str):
    user = user_service.get_user(user_id)
    if not user:
        return {'error': 'Not found'}, 404
    return user.to_dict()

# services/user_service.py
class UserService:
    @handle_service_errors
    def get_user(self, user_id: str) -> Optional[User]:
        return db.users.get(user_id)
```

## Decorators for Cross-Cutting Concerns

❌ Never repeat error handling logic:
```python
async def get_user(user_id: str) -> User:
    try:
        return await db.users.get(user_id)
    except Exception as e:
        log.error(f"Error: {e}")
        raise

async def create_user(data: dict) -> User:
    try:
        return await db.users.create(data)
    except Exception as e:
        log.error(f"Error: {e}")
        raise
```

✅ Always use decorators for common patterns:
```python
@handle_service_errors
async def get_user(user_id: str) -> User:
    return await db.users.get(user_id)

@handle_service_errors
async def create_user(data: dict) -> User:
    return await db.users.create(data)
```

## Type Hints

❌ Never omit type hints:
```python
def process_data(data):
    return data['value']
```

✅ Always include type hints:
```python
from typing import Dict, Any, Optional

def process_data(data: Dict[str, Any]) -> Optional[str]:
    return data.get('value')
```

## Database Queries

❌ Never use raw SQL without parameterization:
```python
query = f"SELECT * FROM users WHERE id = {user_id}"
```

✅ Always use parameterized queries:
```python
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

Or use ORM:
```python
user = User.query.filter_by(id=user_id).first()
```

## Configuration Management

❌ Never hardcode configuration values:
```python
DATABASE_URL = "postgresql://localhost/mydb"
API_KEY = "sk-1234567890"
```

✅ Always use environment variables:
```python
import os
from typing import Optional

DATABASE_URL: Optional[str] = os.getenv('DATABASE_URL')
API_KEY: Optional[str] = os.getenv('API_KEY')

if not DATABASE_URL:
    raise ValueError("DATABASE_URL environment variable is required")
```

## Logging

❌ Never use print statements:
```python
print(f"User {user_id} fetched")
print(f"Error: {error}")
```

✅ Always use structured logging:
```python
import logging

logger = logging.getLogger(__name__)

logger.info("User fetched", extra={'user_id': user_id})
logger.error("Failed to fetch user", extra={
    'user_id': user_id,
    'error': str(error)
})
```

## Async/Await Patterns

❌ Never mix sync and async incorrectly:
```python
def get_user(user_id: str):
    return db.users.get(user_id)  # Blocking call
```

✅ Always use async/await for I/O operations:
```python
async def get_user(user_id: str) -> Optional[User]:
    return await db.users.get(user_id)
```

## Validation

❌ Never trust user input:
```python
def create_user(email: str, name: str):
    db.users.create({'email': email, 'name': name})
```

✅ Always validate input:
```python
from pydantic import BaseModel, EmailStr, validator

class UserCreate(BaseModel):
    email: EmailStr
    name: str
    
    @validator('name')
    def name_must_not_be_empty(cls, v):
        if not v or not v.strip():
            raise ValueError('Name cannot be empty')
        return v.strip()

def create_user(data: UserCreate):
    db.users.create(data.dict())
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mon-amit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mon-amit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
