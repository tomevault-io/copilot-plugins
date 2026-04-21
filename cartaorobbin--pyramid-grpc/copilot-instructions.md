## pyramid-grpc

> This document outlines the development rules and guidelines for the pyramid-grpc project. Following these rules ensures code quality, maintainability, and effective collaboration.

# Development Rules

This document outlines the development rules and guidelines for the pyramid-grpc project. Following these rules ensures code quality, maintainability, and effective collaboration.

## Core Development Principles

### 1. Plan Before Coding ⭐
**ALWAYS plan before writing any code.**

- **Think first, code second**: Before touching any code, take time to understand the problem and design the solution
- **Create a clear plan**: Write down what you're going to do, how you're going to do it, and what the expected outcome is
- **Consider edge cases**: Think through potential issues and how to handle them
- **Review existing code**: Understand how your changes fit into the existing codebase
- **Break down complex tasks**: Split large features into smaller, manageable pieces

### 2. Task Management with Planning Files ⭐
**ALWAYS maintain planning files to track what we did.**

- **Organized structure**: Use `planning/` directory for all planning files
- **Active vs Backlog**: Use `planning/general.md` for CURRENT tasks being worked on, `planning/backlog.md` for planned future tasks
- **Task Flow**: Move tasks from `planning/backlog.md` → `planning/general.md` → `planning/done.md` as they progress
- **Focus Management**: Keep `planning/general.md` focused (max 1-3 active task groups)
- **Create/update files**: Before starting any work, document what you plan to do in the appropriate planning file
- **Track progress**: Update the files as you complete tasks
- **Document decisions**: Record important decisions and why they were made
- **Include completion status**: Mark tasks as TODO, IN PROGRESS, DONE, or BLOCKED
- **Reference issues/PRs**: Link to relevant GitHub issues or pull requests

## Code Quality Standards

### Pre-Development Checklist
Before starting any development work:

- [ ] Read and understand the requirement
- [ ] Create or update appropriate planning file with your planned work
- [ ] Add new tasks to `planning/backlog.md`, move to `planning/general.md` when starting active work
- [ ] Check existing issues and PRs for related work
- [ ] Plan your approach and get feedback if needed
- [ ] Ensure your development environment is set up (`make install`)
- [ ] **Plan test strategy**: Identify what tests need to be written/updated

### Development Workflow

1. **Branch Management**
   ```bash
   git checkout -b feature/descriptive-name
   # or
   git checkout -b bugfix/issue-description
   ```

2. **Code Development**
   - Follow PEP 8 style guidelines
   - Write clear, self-documenting code
   - Add docstrings to all functions and classes
   - Include type hints where appropriate

3. **Testing Requirements**
   - Write tests for all new functionality
   - Ensure existing tests pass: `make test`
   - Run full test suite: `tox`
   - Aim for high test coverage
   - never use class base testcase
   - avoid mocking, only use mock as a last resource
   - we can use pytest responses fixture.

4. **Test Validation Before Task Completion ⭐**
   **NEVER mark a task as DONE without running and verifying tests pass.**
   
   - **Always run tests**: Execute relevant test suites before marking tasks complete
   - **Fix failing tests**: Address any test failures before task completion
   - **Verify no regressions**: Run existing test suites to ensure no breaking changes
   - **Update planning files**: Only mark tasks as DONE after successful test validation
   - **Document test results**: Include test pass/fail status in planning files
   
   ```bash
   # Required before marking any task as DONE
   python -m pytest tests/test_relevant_module.py -v  # New functionality tests
   python -m pytest tests/test_existing_module.py -v  # Regression tests
   make test  # Full test suite (when appropriate)
   ```

5. **Code Quality Checks**
   ```bash
   make check  # Run formatters and linters
   ```

### Documentation Standards

- **Docstrings**: Use Google-style docstrings for all public functions and classes
- **Comments**: Write comments for complex logic, not obvious code
- **README updates**: Update README.md if adding new features or changing setup
- **Type hints**: Use type hints for function parameters and return values

### Commit Standards

- **Commit messages**: Use clear, descriptive commit messages
- **Atomic commits**: Each commit should represent one logical change
- **Pre-commit hooks**: Let pre-commit hooks format your code automatically

Example commit message format:
```
feat: add user authentication system

- Implement JWT token authentication
- Add user login/logout endpoints
- Include password hashing utilities
- Add authentication middleware

Closes #123
```

## Pull Request Guidelines

### Before Creating a PR

- [ ] Update appropriate planning file with completed work
- [ ] **All tests pass locally** (new functionality + regression tests)
- [ ] **Test results documented** in planning files
- [ ] Code quality checks pass (`make check`)
- [ ] Documentation is updated if needed
- [ ] Commit messages are clear and descriptive

### PR Requirements

1. **Title**: Clear, descriptive title
2. **Description**: 
   - What was changed and why
   - Link to related issues
   - Any breaking changes
   - Screenshots if UI changes
3. **Testing**: Describe how the changes were tested
4. **Checklist**: Include a checklist of completed items

### PR Template
```markdown
## Description
Brief description of changes

## Related Issues
Closes #123

## Changes Made
- [ ] Feature/fix 1
- [ ] Feature/fix 2

## Testing
- [ ] Unit tests pass
- [ ] Manual testing completed
- [ ] Edge cases considered

## Documentation
- [ ] Code is documented
- [ ] README updated if needed
- [ ] Planning files updated
```

## File Organization

### Required Files for Each Feature/Bug Fix

1. **Planning files** - Track what you're doing and have done in `planning/` directory
2. **Tests** - In the `tests/` directory  
3. **Documentation** - Update relevant docs

### Planning File Structure

The `planning/` directory contains all task tracking and planning files:

- **`planning/general.md`** - CURRENT/ACTIVE tasks being worked on right now (max 1-3 task groups)
- **`planning/backlog.md`** - PLANNED future tasks, prioritized and ready for development
- **`planning/done.md`** - COMPLETED project tasks for historical reference
- **`planning/[feature-name].md`** - Feature-specific planning and tasks
- **`planning/feature-template.md`** - Template for creating new feature files
- **`planning/README.md`** - Documentation for the planning system

### When to Use Which Planning File

**Use `planning/general.md` for:**
- CURRENT tasks actively being worked on  
- IN PROGRESS development work
- Immediate priority tasks
- Keep focused (max 1-3 active task groups)

**Use `planning/backlog.md` for:**
- PLANNED future tasks and features
- TODO tasks not yet started
- Prioritized development pipeline
- Feature requirements and planning

**Move to `planning/done.md` when:**
- Tasks are marked as DONE ✅
- **Implementation is complete AND all tests pass**
- **Test results are documented in planning files**
- Work is ready for archival reference

**Create `planning/[feature-name].md` for:**
- Features taking more than 1 day
- Complex features with multiple tasks
- Features requiring team coordination
- Features with dependencies or special requirements

### Planning File Template

See `planning/feature-template.md` for the complete template, or use this basic structure:

```markdown
### [YYYY-MM-DD] Task Description

**Status**: TODO | IN PROGRESS | DONE | BLOCKED
**Assigned**: Your Name
**Estimated Time**: X hours
**Related Issue**: #123

#### Plan
- [ ] Task 1: Description
- [ ] Task 2: Description
- [ ] Task 3: Description

#### Progress
- [x] Completed task with notes
- [ ] Current task
- [ ] Future task

#### Decisions Made
- Decision 1: Reasoning
- Decision 2: Reasoning

#### Blockers/Issues
- Issue 1: Description and resolution plan
```

## Dependency Management and Library Usage ⭐

### Core Dependency Tools

#### Poetry - Primary Dependency Manager
**ALWAYS use Poetry for dependency management.**

- **Exclusive use**: Poetry is the ONLY allowed dependency manager
- **No mixing**: Never use pip, pipenv, conda, or other tools alongside Poetry
- **Lock file**: Always commit `poetry.lock` to ensure reproducible builds
- **Version constraints**: Use semantic versioning with appropriate constraints

```bash
# Adding dependencies
poetry add requests              # Production dependency
poetry add --group dev pytest   # Development dependency
poetry add --group docs mkdocs  # Documentation dependency

# Updating dependencies
poetry update                    # Update all dependencies
poetry update requests          # Update specific package
poetry show --outdated         # Check for outdated packages
```

#### Poetry Groups Organization
Organize dependencies into clear groups:

```toml
[tool.poetry.dependencies]
python = ">=3.8,<4.0"
# Core runtime dependencies only

[tool.poetry.group.dev.dependencies]
pytest = "^7.2.0"
pytest-cov = "^4.0.0"
mypy = "^0.981"
pre-commit = "^2.20.0"
tox = "^3.25.1"
black = "^22.0.0"
ruff = "^0.1.0"

[tool.poetry.group.docs.dependencies]
mkdocs = "^1.6.1"
mkdocstrings = "^0.29.1"
mkdocstrings-python = "^1.16.10"

[tool.poetry.group.test.dependencies]
# Additional test-only dependencies if needed
```

### Testing Framework

#### pytest - Primary Test Runner
**ALWAYS use pytest for all testing needs.**

- **Exclusive testing**: pytest is the ONLY allowed test framework
- **No alternatives**: Never use unittest, nose, or other test frameworks
- **Configuration**: Use `pyproject.toml` for pytest configuration
- **Coverage**: Always use pytest-cov for coverage reporting

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_functions = ["test_*"]
addopts = [
    "--cov=pyramid_grpc",
    "--cov-report=term-missing",
    "--cov-report=html",
    "--cov-fail-under=80",
    "--strict-markers",
    "--strict-config",
]
markers = [
    "slow: marks tests as slow",
    "integration: marks tests as integration tests",
    "unit: marks tests as unit tests",
]
```

#### Testing Best Practices
- **File structure**: Mirror source structure in `tests/` directory
- **Test naming**: Use descriptive test names starting with `test_`
- **Fixtures**: Use pytest fixtures for reusable test setup
- **Coverage**: Maintain minimum 80% test coverage
- **Markers**: Use markers to categorize tests

```bash
# Running tests
poetry run pytest                    # All tests
poetry run pytest tests/unit/       # Unit tests only
poetry run pytest -m "not slow"     # Skip slow tests
poetry run pytest --cov-report=html # Generate HTML coverage report
```


#### Core Testing Principles

##### ⚠️ CRITICAL: NO CONDITIONAL LOGIC IN TESTS
**NEVER use conditional logic in tests** - This is a STRICT PROHIBITION:

- **❌ NO `if` statements** - No `if condition:` or `if/else` blocks
- **❌ NO `try/except` blocks** - No exception handling in tests
- **❌ NO loops** - No `for`, `while`, or list comprehensions
- **❌ NO ternary operators** - No `value if condition else other_value`

```python
# ❌ FORBIDDEN - Never do this in tests
def test_bad_example():
    response = make_request()
    if response.status_code == 200:
        assert "success" in response.json
    else:
        assert "error" in response.json

# ✅ CORRECT - Write separate focused tests
def test_successful_request_returns_success():
    response = make_successful_request()
    assert response.status_code == 200
    assert "success" in response.json

def test_failed_request_returns_error():
    response = make_failed_request()
    assert response.status_code == 400
    assert "error" in response.json
```

**Why this rule exists:**
- Tests should be predictable and deterministic
- Each test should verify exactly one behavior
- Conditional logic makes tests complex and harder to debug
- Split complex scenarios into multiple focused tests

**One test, one purpose** - Each test should verify exactly one behavior.

**Simple assertions** - Test what should happen, not what shouldn't happen.

**Use descriptive test names** - Name should describe the scenario and expected outcome.

**Use fixtures for setup** - Don't create test data inside test functions. The fixture should live at conftest.py except for the fixture that setup the pyramid. If the test setup a pyramid please keep the fixture at the same file.

**Check the fixtures** - Before create a new fixture check if a similar fixture exists.


### Documentation Framework

#### MkDocs - Primary Documentation Tool
**ALWAYS use MkDocs for documentation generation.**

- **Exclusive docs**: MkDocs is the ONLY allowed documentation generator
- **No alternatives**: Never use Sphinx, GitBook, or other doc tools
- **Configuration**: Use `mkdocs.yml` for all documentation settings
- **Extensions**: Use mkdocstrings for API documentation from docstrings

```yaml
# mkdocs.yml structure
site_name: Pyramid gRPC
repo_url: https://github.com/tomas_correa/pyramid-grpc
docs_dir: docs
site_dir: site

theme:
  name: material

plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          options:
            docstring_style: google
```

### gRPC Service Development

#### ⭐ CRITICAL: gRPC Service Integration with Pyramid
**When developing gRPC services with Pyramid, follow these patterns.**

**Required Pattern:**
```python
import grpc
from pyramid_grpc.decorators import grpc_service
from pyramid.config import Configurator

@grpc_service
class GreetService:
    def __init__(self, request):
        self.request = request
    
    def SayHello(self, request, context):
        # Access Pyramid request context
        pyramid_request = self.request
        
        # Use Pyramid's dependency injection, security, etc.
        user = pyramid_request.authenticated_userid
        
        return HelloReply(message=f"Hello {request.name}!")
```

**Key Requirements:**
- **Use @grpc_service decorator**: Integrates with Pyramid's request lifecycle
- **Access Pyramid request**: Available as `self.request` in service methods
- **Follow gRPC method signatures**: `(self, request, context)` pattern
- **Return proper gRPC response objects**: Use generated protobuf classes

### Forbidden Libraries and Patterns

#### ⚠️ NEVER USE: Pydantic
**Pydantic is STRICTLY FORBIDDEN in this project.**

- **No Pydantic**: Never add pydantic as a dependency
- **No BaseModel**: Don't use Pydantic's BaseModel or validation
- **Alternatives**: Use dataclasses, TypedDict, or attrs instead
- **Reason**: Architectural decision to avoid dependency complexity

```python
# ❌ FORBIDDEN - Never do this
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

# ✅ ALLOWED - Use dataclasses instead
from dataclasses import dataclass
from typing import TypedDict

@dataclass
class User:
    name: str
    age: int

# Or TypedDict for simple structures
class UserDict(TypedDict):
    name: str
    age: int
```

### Dependency Addition Guidelines

#### Use Poetry Add Command (Not Manual Editing)
**ALWAYS use `poetry add` instead of manually editing pyproject.toml.**

- **Correct approach**: Use `poetry add` to add dependencies
- **Wrong approach**: Never manually edit `[tool.poetry.dependencies]` sections
- **Why**: Poetry handles version resolution, conflict detection, and lock file updates automatically

```bash
# ✅ CORRECT - Use poetry add commands
poetry add requests                    # Production dependency
poetry add --group dev logassert      # Development dependency
poetry add --group docs mkdocs        # Documentation dependency

# ❌ WRONG - Never edit pyproject.toml directly
# Don't manually add lines like: requests = "^2.31.0"
```

#### Before Adding Any Dependency
**ALWAYS follow this checklist before adding new dependencies:**

- [ ] Check if functionality exists in Python standard library
- [ ] Verify the library is actively maintained (recent commits/releases)
- [ ] Check license compatibility with project
- [ ] Ensure it's not on the forbidden list
- [ ] Consider the size and complexity it adds
- [ ] **Use `poetry add` command** (never edit pyproject.toml manually)
- [ ] Document the decision in appropriate planning file

#### Preferred Libraries by Category

**Web Frameworks:**
- ✅ Pyramid (primary framework)
- ❌ FastAPI (explicitly forbidden)
- ❌ Django (too heavy for our use case)
- ❌ Flask (prefer Pyramid)

**gRPC Libraries:**
- ✅ grpcio (core gRPC library)
- ✅ grpcio-tools (protobuf compilation)
- ✅ grpcio-reflection (for development/debugging)
- ✅ grpc-interceptor (for middleware-like functionality)

**Data Handling:**
- ✅ dataclasses (Python standard library)
- ✅ attrs (for complex data structures)
- ✅ TypedDict (for simple dictionaries)
- ❌ Pydantic (explicitly forbidden)

**Testing:**
- ✅ pytest (required)
- ✅ pytest-cov (for coverage)
- ✅ pytest-mock (for mocking)
- ❌ unittest (use pytest instead)
- ❌ nose (deprecated)

**HTTP Clients:**
- ✅ httpx (async-first)
- ✅ requests (synchronous)
- ❌ urllib3 directly (use higher-level clients)

**Date/Time:**
- ✅ datetime (Python standard library)
- ✅ dateutil (for parsing)
- ❌ arrow (prefer standard library)
- ❌ pendulum (unnecessary complexity)

#### Version Constraints Best Practices

```toml
# Good version constraints
requests = "^2.31.0"        # Allow compatible updates
pytest = "^7.2.0"          # Major version locked
python = ">=3.8,<4.0"      # Clear range

# Avoid these patterns
requests = "*"              # Too permissive
pytest = "==7.2.0"         # Too restrictive (blocks security updates)
python = "^3.9"            # Could allow Python 4.x unexpectedly
```

### Development Environment

#### Setup
```bash
make install          # Install dependencies and pre-commit hooks
poetry shell         # Activate virtual environment
```

#### Daily Workflow
```bash
make check          # Format and lint code
make test           # Run tests
tox                 # Test across Python versions (optional locally)
```

#### Dependency Management Commands
```bash
# Check dependencies
poetry show --tree          # Show dependency tree
poetry show --outdated      # Check for updates
poetry check                # Validate pyproject.toml

# Update dependencies
poetry update               # Update all within constraints
poetry update requests      # Update specific package
poetry lock --no-update     # Regenerate lock file

# Audit and security
poetry audit                # Check for known vulnerabilities
```

## Review Process

### Self-Review Checklist
Before requesting review:

- [ ] Code follows project conventions
- [ ] All tests pass
- [ ] Documentation is complete
- [ ] Planning files are updated
- [ ] No debugging code left behind
- [ ] Error handling is appropriate
- [ ] Performance implications considered

### Code Review Guidelines

**For Reviewers:**
- Focus on design, logic, and maintainability
- Suggest improvements, don't just point out problems
- Be constructive and respectful
- Check that planning files reflect the work done

**For Authors:**
- Be open to feedback
- Explain complex decisions in comments or PR description
- Update planning files based on review feedback if scope changes

## Emergency/Hotfix Process

Even for urgent fixes:

1. **Still plan first** - Even 5 minutes of planning can prevent bigger issues
2. **Update planning files** - Document what you're fixing in `planning/general.md` for active work (move to `planning/done.md` when complete)
3. **Create hotfix branch**: `hotfix/critical-issue-description`
4. **Minimal, focused changes** - Fix only what's necessary
5. **Fast-track review** - Get quick review but don't skip testing

## Tools and Resources

### Essential Commands

#### Poetry Commands
```bash
# Environment management
poetry install              # Install all dependencies
poetry shell               # Activate virtual environment
poetry env info            # Show virtual environment info

# Dependency management
poetry add package         # Add production dependency
poetry add --group dev pkg # Add development dependency
poetry remove package      # Remove dependency
poetry update             # Update all dependencies
poetry show --tree        # Show dependency tree
poetry show --outdated    # Check for updates
poetry check              # Validate pyproject.toml
poetry audit              # Security audit
```

#### Testing Commands
```bash
# Basic testing
poetry run pytest                    # Run all tests
poetry run pytest --cov            # Run with coverage
poetry run pytest -v               # Verbose output
poetry run pytest tests/unit/      # Run specific directory
poetry run pytest -k "test_name"   # Run tests matching pattern
poetry run pytest -m "not slow"    # Skip slow tests

# Coverage reporting
poetry run pytest --cov-report=html    # HTML coverage report
poetry run pytest --cov-report=term    # Terminal coverage report
poetry run pytest --cov-fail-under=80  # Fail if coverage below 80%
```

#### Documentation Commands
```bash
# MkDocs commands
poetry run mkdocs serve     # Serve docs locally
poetry run mkdocs build     # Build static docs
poetry run mkdocs gh-deploy # Deploy to GitHub Pages
```

#### Development Workflow Commands
```bash
make help           # Show all available make commands
make install        # Full project setup
make check          # Format and lint code
make test           # Run full test suite
make docs           # Build documentation
tox                 # Test across Python versions
```

### Pre-commit Hooks
The project uses pre-commit hooks to automatically:
- Format code with black
- Sort imports with isort
- Lint with flake8
- Run other quality checks

## Pyramid-gRPC Development Practices

### 🔐 gRPC Security with Pyramid

#### gRPC Interceptors vs Pyramid Security

**IMPORTANT**: When integrating gRPC with Pyramid, you can leverage both gRPC interceptors and Pyramid's security system.

#### The Pyramid-gRPC Way: Security Integration

Pyramid-gRPC uses a hybrid approach combining:

1. **gRPC Interceptors**: Handle gRPC-specific authentication (JWT tokens, metadata)
2. **Pyramid Security**: Leverage Pyramid's ACL system for authorization
3. **Request Context**: Bridge gRPC calls to Pyramid request lifecycle
4. **Service Permissions**: gRPC services can use Pyramid's permission system

```python
# ✅ CORRECT - gRPC Interceptor with Pyramid Security
from grpc_interceptor import ServerInterceptor
from pyramid.security import authenticated_userid

class AuthInterceptor(ServerInterceptor):
    def __init__(self, pyramid_request):
        self.pyramid_request = pyramid_request
    
    def intercept(self, method, request, context, method_name):
        # Extract JWT from gRPC metadata
        jwt_token = self._extract_jwt(context)
        
        # Validate with Pyramid security
        if jwt_token:
            self.pyramid_request.jwt_claims = validate_jwt(jwt_token)
        
        return method(request, context)

# ✅ CORRECT - gRPC Service with Pyramid Security
@grpc_service
class SecureGreetService:
    def __init__(self, request):
        self.request = request
    
    def SayHello(self, request, context):
        # Check Pyramid permissions
        if not self.request.has_permission('greet'):
            context.set_code(grpc.StatusCode.PERMISSION_DENIED)
            context.set_details('Access denied')
            return HelloReply()
        
        user = authenticated_userid(self.request)
        return HelloReply(message=f"Hello {request.name} from {user}!")
```

#### gRPC Service Registration

Register gRPC services with Pyramid configuration:

```python
def includeme(config):
    # Register gRPC services
    config.add_grpc_service(GreetService, 'greet_pb2_grpc', 'GreeterServicer')
    
    # Configure interceptors
    config.add_grpc_interceptor(AuthInterceptor)
    
    # Set up gRPC server
    config.configure_grpc_server(port=50051)
```

### 🧪 Testing gRPC Services

#### Test gRPC Service Integration

```python
import pytest
import grpc
from grpc_testing import server_from_dictionary, strict_real_time
from pyramid.testing import DummyRequest

@pytest.fixture
def grpc_test_server():
    """Create a test gRPC server."""
    services = {
        'greet_pb2_grpc.GreeterServicer': GreetService(DummyRequest())
    }
    return server_from_dictionary(services, strict_real_time())

def test_grpc_service_basic_call(grpc_test_server):
    """Test basic gRPC service call."""
    method = grpc_test_server.unary_unary('SayHello')
    request = HelloRequest(name="World")
    
    response, trailing_metadata, code, details = method.with_call(request)
    
    assert code == grpc.StatusCode.OK
    assert response.message == "Hello World!"

def test_grpc_service_with_pyramid_context(grpc_test_server):
    """Test gRPC service with Pyramid request context."""
    # Set up authenticated request
    pyramid_request = DummyRequest()
    pyramid_request.authenticated_userid = "testuser"
    
    service = GreetService(pyramid_request)
    request = HelloRequest(name="World")
    context = grpc_test_server.unary_unary('SayHello')
    
    response = service.SayHello(request, context)
    
    assert "testuser" in response.message
```

#### Test gRPC Interceptors

```python
def test_auth_interceptor_validates_jwt():
    """Test that auth interceptor properly validates JWT tokens."""
    pyramid_request = DummyRequest()
    interceptor = AuthInterceptor(pyramid_request)
    
    # Mock gRPC context with JWT metadata
    context = MockContext()
    context.invocation_metadata = [('authorization', 'Bearer valid-jwt-token')]
    
    # Should extract and validate JWT
    result = interceptor.intercept(mock_method, mock_request, context, 'SayHello')
    
    assert pyramid_request.jwt_claims is not None
    assert result is not None
```

### 🏗️ gRPC Architecture Patterns

#### Pattern 1: Service-Level Security

```python
# Public gRPC service (no authentication required)
@grpc_service
class PublicGreetService:
    def SayHello(self, request, context):
        return HelloReply(message=f"Hello {request.name}!")

# Authenticated gRPC service
@grpc_service
class AuthenticatedGreetService:
    def __init__(self, request):
        self.request = request
    
    def SayHello(self, request, context):
        if not self.request.authenticated_userid:
            context.set_code(grpc.StatusCode.UNAUTHENTICATED)
            context.set_details('Authentication required')
            return HelloReply()
        
        return HelloReply(message=f"Hello {request.name}!")

# Admin-only gRPC service
@grpc_service
class AdminGreetService:
    def __init__(self, request):
        self.request = request
    
    def SayHello(self, request, context):
        if not self.request.has_permission('admin'):
            context.set_code(grpc.StatusCode.PERMISSION_DENIED)
            context.set_details('Admin access required')
            return HelloReply()
        
        return HelloReply(message=f"Admin greeting for {request.name}!")
```

#### Pattern 2: Method-Level Security

```python
@grpc_service
class UserService:
    def __init__(self, request):
        self.request = request
    
    def GetUser(self, request, context):
        # Public method - no auth required
        return UserResponse(name=request.user_id)
    
    def UpdateUser(self, request, context):
        # Requires authentication
        if not self.request.authenticated_userid:
            context.set_code(grpc.StatusCode.UNAUTHENTICATED)
            return UserResponse()
        
        # Check ownership or admin
        current_user = self.request.authenticated_userid
        if current_user != request.user_id and not self.request.has_permission('admin'):
            context.set_code(grpc.StatusCode.PERMISSION_DENIED)
            return UserResponse()
        
        # Proceed with update
        return UserResponse(name=request.user_id, updated=True)
    
    def DeleteUser(self, request, context):
        # Admin only
        if not self.request.has_permission('admin'):
            context.set_code(grpc.StatusCode.PERMISSION_DENIED)
            return UserResponse()
        
        return UserResponse(deleted=True)
```

#### Pattern 3: Interceptor-Based Security

```python
class RoleBasedInterceptor(ServerInterceptor):
    def __init__(self, pyramid_request):
        self.pyramid_request = pyramid_request
    
    def intercept(self, method, request, context, method_name):
        # Extract required role from method metadata
        required_role = self._get_required_role(method_name)
        
        if required_role:
            user_roles = self.pyramid_request.effective_principals
            if f'role:{required_role}' not in user_roles:
                context.set_code(grpc.StatusCode.PERMISSION_DENIED)
                context.set_details(f'Role {required_role} required')
                return None
        
        return method(request, context)
```

### 📋 gRPC Security Best Practices

#### 1. Always Validate Authentication in gRPC Services

```python
# ✅ GOOD - Explicit authentication check
@grpc_service
class SecureService:
    def __init__(self, request):
        self.request = request
    
    def SecureMethod(self, request, context):
        if not self.request.authenticated_userid:
            context.set_code(grpc.StatusCode.UNAUTHENTICATED)
            context.set_details('Authentication required')
            return EmptyResponse()
        
        return ProcessedResponse(data="secure data")

# ❌ BAD - No authentication check
@grpc_service
class InsecureService:
    def SecureMethod(self, request, context):
        # No auth check - security vulnerability
        return ProcessedResponse(data="secure data")
```

#### 2. Use Specific gRPC Status Codes

```python
# ✅ GOOD - Specific, meaningful status codes
def check_permissions(self, context, required_permission):
    if not self.request.authenticated_userid:
        context.set_code(grpc.StatusCode.UNAUTHENTICATED)
        context.set_details('Authentication required')
        return False
    
    if not self.request.has_permission(required_permission):
        context.set_code(grpc.StatusCode.PERMISSION_DENIED)
        context.set_details(f'Permission {required_permission} required')
        return False
    
    return True

# ❌ BAD - Generic error codes
def bad_check_permissions(self, context):
    if not self.request.authenticated_userid:
        context.set_code(grpc.StatusCode.INTERNAL)  # Too generic
        return False
```

#### 3. Secure by Default with Interceptors

```python
# ✅ GOOD - Default authentication interceptor
class DefaultAuthInterceptor(ServerInterceptor):
    def intercept(self, method, request, context, method_name):
        # Require auth by default, unless explicitly public
        if not self._is_public_method(method_name):
            if not self._validate_auth(context):
                context.set_code(grpc.StatusCode.UNAUTHENTICATED)
                return None
        
        return method(request, context)

# ❌ BAD - No default security
class NoSecurityInterceptor(ServerInterceptor):
    def intercept(self, method, request, context, method_name):
        # No security checks - everything is public by default
        return method(request, context)
```

#### 4. Test gRPC Security Boundaries

```python
def test_grpc_security_boundaries():
    """Test that gRPC security works correctly at boundaries."""
    # Test unauthenticated access (should return UNAUTHENTICATED)
    # Test authenticated access (should work)
    # Test wrong role access (should return PERMISSION_DENIED)
    # Test privilege escalation attempts (should fail)
    pass
```

### 🐛 Common gRPC Security Pitfalls

#### 1. Forgetting Authentication Checks

```python
# ❌ WRONG - No authentication check in gRPC service
@grpc_service
class UserService:
    def GetUserData(self, request, context):
        # Security vulnerability - no auth check
        return UserData(sensitive_info="secret")

# ✅ RIGHT - Always check authentication
@grpc_service
class UserService:
    def __init__(self, request):
        self.request = request
    
    def GetUserData(self, request, context):
        if not self.request.authenticated_userid:
            context.set_code(grpc.StatusCode.UNAUTHENTICATED)
            return UserData()
        return UserData(sensitive_info="secret")
```

#### 2. Using Generic Error Codes

```python
# ❌ WRONG - Generic error codes
def handle_error(self, context):
    context.set_code(grpc.StatusCode.INTERNAL)  # Too generic
    context.set_details('Error')  # Not helpful

# ✅ RIGHT - Specific error codes
def handle_auth_error(self, context):
    context.set_code(grpc.StatusCode.UNAUTHENTICATED)
    context.set_details('Valid JWT token required')

def handle_permission_error(self, context, permission):
    context.set_code(grpc.StatusCode.PERMISSION_DENIED)
    context.set_details(f'Permission {permission} required')
```

#### 3. Not Validating gRPC Metadata

```python
# ❌ WRONG - Trusting gRPC metadata without validation
def extract_user_id(self, context):
    metadata = dict(context.invocation_metadata())
    return metadata.get('user-id')  # No validation

# ✅ RIGHT - Validate and sanitize metadata
def extract_user_id(self, context):
    metadata = dict(context.invocation_metadata())
    user_id = metadata.get('user-id')
    
    if not user_id or not self._is_valid_user_id(user_id):
        context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
        context.set_details('Invalid user ID in metadata')
        return None
    
    return user_id
```

### 🔄 Migration from REST to gRPC

If you're migrating from REST APIs to gRPC:

#### Before (REST)

```python
# REST endpoint with Pyramid
@view_config(route_name='api_user', request_method='GET', renderer='json')
def get_user(request):
    if not request.authenticated_userid:
        raise HTTPUnauthorized()
    
    user_id = request.matchdict['user_id']
    return {'user_id': user_id, 'name': 'John'}
```

#### After (gRPC)

```python
# gRPC service with Pyramid integration
@grpc_service
class UserService:
    def __init__(self, request):
        self.request = request
    
    def GetUser(self, request, context):
        if not self.request.authenticated_userid:
            context.set_code(grpc.StatusCode.UNAUTHENTICATED)
            context.set_details('Authentication required')
            return UserResponse()
        
        return UserResponse(user_id=request.user_id, name='John')
```

### 📚 Additional Resources

- [Pyramid Documentation](https://docs.pylonsproject.org/projects/pyramid/en/latest/)
- [gRPC Python Documentation](https://grpc.io/docs/languages/python/)
- [Protocol Buffers Guide](https://developers.google.com/protocol-buffers/docs/pythontutorial)

---

## Questions or Issues?

- Check existing issues first
- Create detailed issue reports
- Ask for help early rather than struggling alone
- Update planning files when blocked with explanation

---

**Remember**: These rules exist to make our development process smoother and our code better. When in doubt, plan first and document what you're doing in the appropriate planning file! 

### gRPC Testing Patterns

#### ⭐ CRITICAL: Use pytest-grpc for gRPC Testing
**ALWAYS use pytest-grpc fixtures for testing gRPC services.**

```python
# ✅ CORRECT - Use pytest-grpc fixtures
import pytest
from pytest_grpc import grpc_channel, grpc_server

@pytest.fixture
def grpc_add_to_server():
    from tests.services.greet_pb2_grpc import add_GreeterServicer_to_server
    return add_GreeterServicer_to_server

@pytest.fixture
def grpc_servicer():
    from tests.services.greet import GreetService
    from pyramid.testing import DummyRequest
    return GreetService(DummyRequest())

def test_grpc_service(grpc_channel, grpc_stub):
    """Test gRPC service with proper fixtures."""
    from tests.services.greet_pb2 import HelloRequest
    
    request = HelloRequest(name="World")
    response = grpc_stub.SayHello(request)
    
    assert response.message == "Hello World!"
```

#### Testing gRPC Services with Pyramid Context
**Integrate Pyramid request context into gRPC service tests.**

```python
@pytest.fixture
def authenticated_grpc_servicer():
    """Create gRPC servicer with authenticated Pyramid request."""
    from pyramid.testing import DummyRequest
    from tests.services.greet import GreetService
    
    request = DummyRequest()
    request.authenticated_userid = "testuser"
    request.effective_principals = ['system.Everyone', 'system.Authenticated', 'role:user']
    
    return GreetService(request)

def test_authenticated_grpc_call(grpc_channel, authenticated_grpc_servicer):
    """Test gRPC service with authenticated user."""
    from tests.services.greet_pb2_grpc import GreeterStub
    from tests.services.greet_pb2 import HelloRequest
    
    stub = GreeterStub(grpc_channel)
    request = HelloRequest(name="World")
    response = stub.SayHello(request)
    
    assert "testuser" in response.message
```

#### Available Fixtures
- `grpc_channel` - gRPC test channel from pytest-grpc
- `grpc_server` - gRPC test server from pytest-grpc  
- `grpc_stub` - Auto-generated stub for your service
- `pyramid_request` - Pyramid request for service context 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cartaorobbin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
