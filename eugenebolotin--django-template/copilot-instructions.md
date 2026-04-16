## django-template

> - **Use `uv`** as the package manager (fast replacement for `pip`)

# Django Template Project Rules

## Dependency Management

- **Use `uv`** as the package manager (fast replacement for `pip`)
- All dependencies are managed through `pyproject.toml`
- To install dependencies use: `uv sync`
- To run commands use: `uv run <command>`
- Example: `uv run manage.py runserver`

## Django Application Architecture

### Application Structure

Each Django application should have the following structure:

```
app_name/
    __init__.py
    models.py
    admin.py
    views.py          # Thin layer for handling HTTP requests
    urls.py
    services/         # Folder with business logic
        __init__.py
        service1.py   # Individual services with business logic
        service2.py
    tests/            # Folder with tests
        __init__.py
        test_models.py
        test_service1.py   # Separate test file for each service
        test_service2.py
        test_views.py
```

### Separation of Concerns

#### Views (views.py)
- **Thin layer** responsible for:
  - Request deserialization (input data validation)
  - Response serialization (output data formatting)
  - Calling the appropriate function from services
- Views MUST NOT contain business logic
- Views should be as simple as possible and delegate all logic to services
- **Use serializers or type specs** (depending on the framework used) for processing input parameters and formatting results:
  - Django REST Framework: use DRF serializers
  - django-ninja: use Pydantic models or type specs
  - Other frameworks: use appropriate serialization mechanisms

#### Services (services/ folder)
- **All business logic** is located in services
- Each service is a separate Python file with functions/classes
- Services can use Django models but should not work directly with HTTP requests/responses
- Services should be reusable and independent of the presentation layer (views)

### Code Structure Example

```python
# app_name/services/user_service.py
def create_user(email: str, password: str) -> User:
    """Business logic for creating a user"""
    # All logic here
    pass

# Example with Django REST Framework
# app_name/views.py (DRF)
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import UserCreateSerializer, UserResponseSerializer
from .services.user_service import create_user

@api_view(['POST'])
def create_user_view(request):
    """Thin layer for creating a user"""
    # Input data validation using serializer
    serializer = UserCreateSerializer(data=request.data)
    if not serializer.is_valid():
        return Response(serializer.errors, status=400)
    
    # Call business logic
    user = create_user(
        email=serializer.validated_data['email'],
        password=serializer.validated_data['password']
    )
    
    # Response serialization using serializer
    response_serializer = UserResponseSerializer(user)
    return Response(response_serializer.data, status=201)

# Example with django-ninja
# app_name/schemas.py
from pydantic import BaseModel, EmailStr

class UserCreateSchema(BaseModel):
    email: EmailStr
    password: str

class UserResponseSchema(BaseModel):
    id: int
    email: str

# app_name/views.py (django-ninja)
from ninja import Router
from .schemas import UserCreateSchema, UserResponseSchema
from .services.user_service import create_user

router = Router()

@router.post('/users', response=UserResponseSchema)
def create_user_view(request, data: UserCreateSchema):
    """Thin layer for creating a user"""
    # Input data validation using Pydantic schema (automatic)
    # Call business logic
    user = create_user(
        email=data.email,
        password=data.password
    )
    
    # Response serialization using Pydantic schema (automatic)
    return UserResponseSchema(id=user.id, email=user.email)
```

## Testing

- Tests are implemented in the `tests/` folder of each Django application
- Test structure should reflect the application structure:
  - `test_models.py` - tests for models
  - `test_<service_name>.py` - separate test file for each service (e.g., `test_user_service.py` for `services/user_service.py`)
  - `test_views.py` - tests for views
- **A separate test file is created for each service** - this simplifies navigation and test maintenance
- Use standard Django testing tools (unittest.TestCase or pytest)

## Commit Messages

### Conventional Commits

When forming commit messages, follow the **Conventional Commits** principles:
https://www.conventionalcommits.org/ru/v1.0.0-beta.2/

### Commit message format

```
<type>[optional scope]: <short description>

[optional body]

[optional footer]
```

### Commit types

- `fix:` - bug fix (corresponds to PATCH in SemVer)
- `feat:` - adding new functionality (corresponds to MINOR in SemVer)
- `BREAKING CHANGE:` - breaking change (corresponds to MAJOR in SemVer)
- Other types: `chore:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, `improvement:`

### Scope

Scope can be specified to describe the context of the commit:
- `feat(auth): add password reset functionality`
- `fix(api): resolve serialization error`

### Forming commit messages

When it's necessary to form a commit message:

1. **Review the last 50 commits**: `git log --oneline -50`
2. **Study code changes**: `git diff`
3. **Determine the type of change** (fix, feat, refactor, etc.)
4. **Determine the scope** - Django application name or module name
5. **Formulate a short description** in Russian or English
6. **Add commit body if necessary** with detailed description

### Commit message examples

```
feat(auth): add user registration endpoint

fix(api): resolve validation error in user serializer

refactor(services): extract user creation logic to separate service

docs: update README with installation instructions

BREAKING CHANGE: change user model primary key from int to UUID
```

## Code Style

- Use `ruff` for code formatting and linting
- Follow PEP 8 considering settings in `ruff.toml`
- Maximum line length: 120 characters
- Use single quotes for strings

## Additional Recommendations

- Always check code with linter before committing
- Write tests for new functionality
- Document complex business logic in docstrings
- Use type hints to improve code readability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eugenebolotin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
