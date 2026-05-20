## development-workflow

> - Python 3.9+ (as specified in [pyproject.toml](mdc:pyproject.toml))

# Development Workflow for Kiwoom REST API

## Development Environment Setup

### Prerequisites
- Python 3.9+ (as specified in [pyproject.toml](mdc:pyproject.toml))
- Poetry for dependency management
- Git for version control

### Initial Setup
```bash
# Clone repository
git clone <repository-url>
cd kiwoom-rest-api

# Install dependencies
poetry install

# Activate virtual environment
poetry shell

# Set up environment variables
cp .env.example .env
# Edit .env with your API credentials
```

## Development Process

### 1. Understanding the Codebase
- Review [project-structure.mdc](mdc:.cursor/rules/project-structure.mdc) for overall architecture
- Study [api-development-patterns.mdc](mdc:.cursor/rules/api-development-patterns.mdc) for implementation patterns
- Reference [korean-stock-api-reference.mdc](mdc:.cursor/rules/korean-stock-api-reference.mdc) for API details

### 2. Adding New API Methods

#### Step 1: Identify the API
- Determine the API ID and module
- Review existing similar methods in the target module
- Understand the API parameters and response structure

#### Step 2: Implement the Method
- Follow the standard method pattern from [api-development-patterns.mdc](mdc:.cursor/rules/api-development-patterns.mdc)
- Use the correct module file (e.g., [src/kiwoom_rest_api/koreanstock/account.py](mdc:src/kiwoom_rest_api/koreanstock/account.py))
- Include comprehensive Korean documentation
- Add proper type hints and parameter validation

#### Step 3: Add Test Case
- Create or update test file in [tests/koreanstock/](mdc:tests/koreanstock/)
- Follow the test pattern with `print_result()` function
- Use `print_result=False` for clean output
- Test with real API credentials

#### Step 4: Verify Implementation
```bash
# Run tests
poetry run pytest tests/koreanstock/test_account.py -v

# Run specific test
poetry run python tests/koreanstock/test_account.py
```

### 3. Code Quality Standards

#### Documentation
- Include Korean descriptions for all parameters and return fields
- Provide clear usage examples in docstrings
- Document API IDs and their purposes
- Use consistent formatting across all methods

#### Code Style
- Follow PEP 8 guidelines
- Use type hints for all parameters
- Maintain consistent naming conventions
- Keep methods focused and single-purpose

#### Error Handling
- Use the base class `_execute_request` method
- Return raw API responses for caller flexibility
- Document expected error codes and messages

### 4. Testing Strategy

#### Test Organization
- Group tests by module functionality
- Use descriptive test names
- Include both success and error scenarios
- Test with various parameter combinations

#### Test Data
- Use real stock codes (e.g., "005930" for 삼성전자)
- Use valid date formats ("YYYYMMDD")
- Test with different market codes (KRX, NXT, etc.)

#### Environment Management
- Use `.env` file for API credentials
- Never commit sensitive credentials
- Use mock data for unit tests when appropriate

### 5. Version Control

#### Branch Strategy
- Create feature branches for new API implementations
- Use descriptive branch names (e.g., `feature/kt00012-api`)
- Keep branches focused on single features

#### Commit Messages
- Follow the format specified in [git-commit-rules.mdc](mdc:.cursor/rules/git-commit-rules.mdc)
- Include file and line counts
- Use appropriate commit types
- Provide detailed descriptions

#### Pull Request Process
- Include comprehensive description of changes
- Reference related API documentation
- Ensure all tests pass
- Update documentation if needed

### 6. Deployment and Release

#### Version Management
- Update version in [pyproject.toml](mdc:pyproject.toml)
- Follow semantic versioning
- Update [README.md](mdc:README.md) with new features

#### Release Process
```bash
# Build package
poetry build

# Publish to PyPI
poetry publish

# Create git tag
git tag v0.1.11
git push origin v0.1.11
```

### 7. Maintenance

#### Regular Tasks
- Update dependencies with `poetry update`
- Review and update documentation
- Monitor API changes from Kiwoom
- Address user feedback and issues

#### Code Review
- Review new implementations against established patterns
- Ensure consistency across modules
- Verify test coverage
- Check documentation quality

## Common Development Scenarios

### Adding New API Method
1. Study existing similar methods
2. Implement following the standard pattern
3. Add comprehensive documentation
4. Create test case
5. Verify functionality
6. Commit with proper message format

### Fixing Bugs
1. Identify the issue and affected module
2. Implement fix following existing patterns
3. Add test case to prevent regression
4. Update documentation if needed
5. Commit with [Fix] type

### Updating Documentation
1. Review existing documentation
2. Update relevant files
3. Ensure consistency across modules
4. Test examples if applicable
5. Commit with [Docs] type

### Refactoring Code
1. Identify areas for improvement
2. Maintain existing functionality
3. Update tests if needed
4. Ensure backward compatibility
5. Commit with [Refactor] type

---
> Source: [bamjun/kiwoom-rest-api](https://github.com/bamjun/kiwoom-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
