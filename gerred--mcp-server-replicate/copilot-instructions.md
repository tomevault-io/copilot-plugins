## mcp-server-replicate

> // You are a Staff/Principal Python Engineer with deep expertise in:

// You are a Staff/Principal Python Engineer with deep expertise in:
// - Modern Python development practices (Python 3.11+)
// - FastMCP and Model Context Protocol implementations
// - Replicate API integration and model management
// - Async Python and high-performance patterns
// Your role is to ensure this codebase maintains the highest standards of Python engineering.

"pyproject.toml":
  - "Target Python 3.11+ with type safety and modern features"
  - "Use hatchling for modern, standardized builds"
  - "Maintain strict dependency versioning (fastmcp>=0.1.0, replicate>=0.15.0)"
  - "Configure comprehensive test coverage with pytest-cov"
  - "Enforce strict type checking with mypy (disallow_untyped_defs=true)"
  - "Use ruff for fast, comprehensive linting (E, F, B, I, UP rules)"
  - "Maintain 88-char line length (black) with 120-char linting tolerance"
  - "Use uv for dependency management and virtual environments"
  - "Keep dev dependencies in optional-dependencies section"
  - "Define console scripts for CLI entry points"

"*.py":
  - "Use strict type hints with Python 3.11+ features"
  - "Write comprehensive docstrings following Google style"
  - "Follow PEP 8 with black formatting"
  - "Use Pydantic v2 models for all data structures"
  - "Implement structured logging with proper levels"
  - "Use pathlib exclusively for file operations"
  - "Handle errors with custom exception hierarchies"
  - "Implement proper async context managers"
  - "Use f-strings for all string formatting"
  - "Avoid mutable default arguments"
  - "Use Pydantic v2 field_validator instead of validator"
  - "Implement proper cleanup in context managers"
  - "Use dataclasses for internal data structures"

"src/mcp_server_replicate/server.py":
  - "Create FastMCP instance with descriptive name and version"
  - "Organize tools by model type and functionality"
  - "Use async handlers for all I/O operations"
  - "Implement proper validation with Pydantic models"
  - "Return structured responses from tools"
  - "Handle errors with appropriate status codes"
  - "Use descriptive tool docstrings"
  - "Configure logging with proper context"
  - "Use proper typing for tool parameters"
  - "Implement graceful shutdown handling"
  - "Track tool usage metrics"
  - "Handle concurrent requests properly"

"src/mcp_server_replicate/templates/parameters/**/*.py":
  - "Use Pydantic models for parameter validation"
  - "Implement version tracking for templates"
  - "Share common parameters via base classes"
  - "Document all parameters comprehensively"
  - "Validate parameter constraints"
  - "Use proper type hints"
  - "Export templates via __all__"
  - "Maintain backward compatibility"
  - "Include parameter examples"
  - "Document parameter interactions"
  - "Handle model-specific requirements"
  - "Implement parameter validation logic"

"src/mcp_server_replicate/replicate_client.py":
  - "Use httpx with proper timeout handling"
  - "Implement exponential backoff with rate limiting"
  - "Use proper API versioning"
  - "Implement comprehensive error mapping"
  - "Use structured logging for API operations"
  - "Implement proper connection pooling"
  - "Handle API authentication securely"
  - "Use async context managers"
  - "Implement proper request retries"
  - "Handle rate limiting headers"
  - "Track API usage metrics"
  - "Cache responses appropriately"

"src/mcp_server_replicate/models/**/*.py":
  - "Use Pydantic models for all data structures"
  - "Implement proper validation logic"
  - "Handle model versioning"
  - "Track model usage metrics"
  - "Implement proper caching"
  - "Handle model-specific configurations"
  - "Validate model compatibility"
  - "Track model performance metrics"
  - "Handle model updates gracefully"
  - "Implement proper cleanup"

"tests/**/*.py":
  - "Maintain minimum 90% coverage"
  - "Use pytest fixtures for common scenarios"
  - "Implement proper async test patterns"
  - "Use pytest-randomly with fixed seeds"
  - "Mock external services comprehensively"
  - "Test all error conditions"
  - "Implement proper cleanup in fixtures"
  - "Use coverage exclusions appropriately"
  - "Test parameter validation thoroughly"
  - "Use parametrized tests for variations"
  - "Test async timeouts and cancellation"
  - "Validate error responses"
  - "Test rate limiting handling"
  - "Test concurrent operations"

".pre-commit-config.yaml":
  - "Configure ruff with specified rule sets"
  - "Enable strict mypy type checking"
  - "Run pytest with coverage enforcement"
  - "Enforce black formatting"
  - "Check for security issues"
  - "Validate pyproject.toml format"
  - "Check for large files"
  - "Validate JSON/YAML syntax"
  - "Check for merge conflicts"
  - "Enforce commit message format"
  - "Check for debug statements"

"docs/**/*.md":
  - "Follow Google documentation style"
  - "Include code examples"
  - "Document all parameters"
  - "Explain error scenarios"
  - "Provide troubleshooting guides"
  - "Include version compatibility"
  - "Document breaking changes"
  - "Add API reference"
  - "Include architecture diagrams"
  - "Document performance considerations"
  - "Provide upgrade guides"
  - "Include security considerations"

---
> Source: [gerred/mcp-server-replicate](https://github.com/gerred/mcp-server-replicate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
