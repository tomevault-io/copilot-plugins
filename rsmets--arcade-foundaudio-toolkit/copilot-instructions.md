## development-workflow

> Development workflow and quality standards


# Development Workflow and Quality Standards

## Package Management with uv
- Use `uv sync --active --all-extras --no-sources` for installation
- Development commands via [Makefile](mdc:foundaudio/Makefile):
  - `make install` - Install dependencies and pre-commit hooks
  - `make test` - Run pytest with coverage
  - `make check` - Run pre-commit and mypy
  - `make build` - Build wheel file

## Code Quality Standards
- **Type Checking**: mypy configuration in [pyproject.toml](mdc:foundaudio/pyproject.toml)
- **Linting**: Trunk metalinting for comprehensive code quality
- **Testing**: Minimum coverage requirements with pytest-cov
- **Pre-commit**: Automated quality checks on commit

## Arcade.dev Specific Workflows
### Development Server
```bash
uv run arcade serve --reload  # Auto-reload during development
# Server available at http://localhost:8002
```

### Deployment
```bash
uv run arcade deploy  # Deploy to Arcade's managed infrastructure
uv run arcade deploy status  # Check deployment status
```

### Evaluation
```bash
arcade eval  # Run evaluation suite (currently has issues - see GitHub issues)
```

## File Organization Principles
- Keep tools in separate files under `foundaudio/tools/`
- Export tools in `__init__.py` files for proper registration
- Maintain parallel test structure matching source code
- Use descriptive filenames that match tool functionality

## Documentation Standards
- Comprehensive README with setup, usage, and examples
- Inline documentation for complex business logic
- Type hints for all function signatures
- Docstrings following Google/NumPy style

## Professional Development Practices
- **Foundation First**: Establish solid practices before adding features
- **Test-Driven**: Write tests for all functionality with proper mocking
- **Error Handling**: Distinguish between user errors and system errors
- **Security**: Proper secret management patterns even for public keys

---
> Source: [rsmets/arcade-foundaudio-toolkit](https://github.com/rsmets/arcade-foundaudio-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
