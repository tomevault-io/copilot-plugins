## kotorscript-session-manager

> Python best practices and patterns for modern software development with Flask and SQLite


## ⚠️ CRITICAL: Git Commit Requirement ⚠️

**MANDATORY FOR ALL AI AGENTS**: After making ANY changes to files in this repository, you MUST:

1. **Stage all modified files**: `git add <file1> <file2> ...` (or `git add .` for all changes)
2. **Commit with a descriptive message**: `git commit -m "type: descriptive message"`
3. **Verify the commit succeeded**: Check that `git commit` returns successfully with no errors

**This is NOT optional.** Do NOT summarize changes without committing them. Do NOT skip this step. Every code change, file modification, or configuration update MUST be committed before you consider the task complete.

**Commit message format**: Use conventional commits format:

- `fix:` for bug fixes
- `feat:` for new features
- `refactor:` for code refactoring
- `docs:` for documentation changes
- `chore:` for maintenance tasks
- `test:` for test changes

Example: `git commit -m "fix: add healthcheck and self-healing to critical infrastructure services"`

---

## ⚠️ CRITICAL: Healthcheck Requirements ⚠️

**MANDATORY**: ALL Docker services MUST have comprehensive, exhaustive healthchecks. 

**NEVER:**
- ❌ Disable healthchecks
- ❌ Comment out healthchecks
- ❌ Use simplified TCP-only checks (nc -z)
- ❌ Remove healthchecks to "fix" issues

**ALWAYS:**
- ✅ Use full HTTP/HTTPS endpoint checks that validate the service actually works
- ✅ Include proper intervals, timeouts, retries, and start_period
- ✅ Test actual service functionality, not just port availability
- ✅ Add deunhealth labels for self-healing: `deunhealth.restart.on.unhealthy: "true"`
- ✅ If a container lacks health check tools (curl/wget), install them in the healthcheck command
- ✅ Use comprehensive healthchecks like: `wget --spider`, `curl -f`, API endpoint validation

**Example of PROPER healthcheck:**
```yaml
healthcheck:
  test: ["CMD-SHELL", "wget --no-verbose --tries=1 --spider http://127.0.0.1:8080/health || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 30s
```

**If "fix" means disabling healthchecks, you're doing it WRONG. Fix the actual problem.**

---

# Python Best Practices

## Project Structure

- Use src-layout with `src/your_package_name/`
- Place tests in `tests/` directory parallel to `src/`
- Keep configuration in `config/` or as environment variables
- Store requirements in `requirements.txt` or `pyproject.toml`
- Place static files in `static/` directory
- Use `templates/` for Jinja2 templates

## Code Style

- Follow Black code formatting
- Use isort for import sorting
- Follow PEP 8 naming conventions:
  - snake_case for functions and variables
  - PascalCase for classes
  - UPPER_CASE for constants
- Maximum line length of 88 characters (Black default)
- Use absolute imports over relative imports

## Type Hints

- Use type hints for all function parameters and returns
- Import types from `typing` module
- Use `Optional[Type]` instead of `Type | None`
- Use `TypeVar` for generic types
- Define custom types in `types.py`
- Use `Protocol` for duck typing

## Flask Structure

- Use Flask factory pattern
- Organize routes using Blueprints
- Use Flask-SQLAlchemy for database
- Implement proper error handlers
- Use Flask-Login for authentication
- Structure views with proper separation of concerns

## Database

- Use SQLAlchemy ORM
- Implement database migrations with Alembic
- Use proper connection pooling
- Define models in separate modules
- Implement proper relationships
- Use proper indexing strategies

## Authentication

- Use Flask-Login for session management
- Implement Google OAuth using Flask-OAuth
- Hash passwords with bcrypt
- Use proper session security
- Implement CSRF protection
- Use proper role-based access control

## API Design

- Use Flask-RESTful for REST APIs
- Implement proper request validation
- Use proper HTTP status codes
- Handle errors consistently
- Use proper response formats
- Implement proper rate limiting

## Testing

- Use pytest for testing
- Write tests for all routes
- Use pytest-cov for coverage
- Implement proper fixtures
- Use proper mocking with pytest-mock
- Test all error scenarios

## Security

- Use HTTPS in production
- Implement proper CORS
- Sanitize all user inputs
- Use proper session configuration
- Implement proper logging
- Follow OWASP guidelines

## Performance

- Use proper caching with Flask-Caching
- Implement database query optimization
- Use proper connection pooling
- Implement proper pagination
- Use background tasks for heavy operations
- Monitor application performance

## Error Handling

- Create custom exception classes
- Use proper try-except blocks
- Implement proper logging
- Return proper error responses
- Handle edge cases properly
- Use proper error messages

## Documentation

- Use Google-style docstrings
- Document all public APIs
- Keep README.md updated
- Use proper inline comments
- Generate API documentation
- Document environment setup

## Development Workflow

- Use virtual environments (venv)
- Implement pre-commit hooks
- Use proper Git workflow
- Follow semantic versioning
- Use proper CI/CD practices
- Implement proper logging

## Dependencies

- Pin dependency versions
- Use requirements.txt for production
- Separate dev dependencies
- Use proper package versions
- Regularly update dependencies
- Check for security vulnerabilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/th3w1zard1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
