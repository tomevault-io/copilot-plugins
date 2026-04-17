## data-pipeline-python-vm

> - **Data Fetching**: Use aiohttp for HTTP requests

# Cursor Rules for Data Pipeline Backend

# Common Patterns
- **Data Fetching**: Use aiohttp for HTTP requests
- **Caching**: Use Redis for temporary storage
- **Rate Limiting**: Implement delays between requests
- **Error Recovery**: Implement retry logic
- **Monitoring**: Add metrics for workflow performance

# Project Structure

backend/
├── src/
│   ├── api/          # FastAPI application
│   ├── core/         # Core functionality
│   ├── workflows/    # Workflow implementations
│   │   └── examples/ # Example workflows
│   └── utils/        # Utility functions
├── tests/            # Test files
├── docker-compose.yml
└── setup.sh

# Code Style
- Use 4 spaces for indentation
- Maximum line length: 100 characters
- Use double quotes for strings
- Use snake_case for variables and functions
- Use PascalCase for classes
- Use UPPER_CASE for constants

# Python Specific
- Use type hints for all function parameters and return values
- Use async/await for I/O operations
- Use dataclasses or Pydantic models for data structures
- Use pathlib for file paths
- Use logging instead of print statements
- Use f-strings for string formatting
- Use list/dict comprehensions when appropriate
- Use context managers for resource management

# Imports
- Group imports in the following order:
  1. Standard library
  2. Third-party packages
  3. Local application imports
- Use absolute imports for local modules
- Avoid wildcard imports (*)

# Documentation
- Use docstrings for all modules, classes, and functions
- Follow Google style docstring format
- Include type information in docstrings
- Document exceptions that may be raised
- Include examples for complex functions

# Testing
- Write tests for all new features
- Use pytest for testing
- Use pytest-asyncio for async tests
- Use pytest-cov for coverage
- Aim for >80% test coverage
- Use meaningful test names
- Use fixtures for common test setup
- Mock external dependencies

# Error Handling
- Use specific exception types
- Include meaningful error messages
- Use try/except blocks appropriately
- Log errors with context
- Handle cleanup in finally blocks

# Workflow Development
- Inherit from BaseWorkflow
- Implement process() method
- Use WorkflowResult for returns
- Include proper error handling
- Add logging statements
- Use configuration from config.json
- Implement cleanup() if needed

# Redis Usage
- Use QueueManager for Redis operations
- Set appropriate queue lengths
- Handle connection errors
- Implement proper cleanup
- Use meaningful queue names

# Security
- Never commit secrets
- Use environment variables for sensitive data
- Validate all input data
- Sanitize user input
- Use secure defaults

# Performance
- Use async/await for I/O
- Implement proper caching
- Use connection pooling
- Monitor memory usage
- Profile slow operations

# Git
- Use meaningful commit messages
- Keep commits focused
- Use feature branches
- Review code before merging
- Keep main branch stable

# CI/CD
- All tests must pass
- No linting errors
- Maintain test coverage
- Document deployment process
- Use semantic versioning

# Monitoring
- Add logging statements
- Use structured logging
- Include metrics
- Monitor queue sizes
- Track execution times

# Dependencies
- Pin dependency versions
- Keep dependencies updated
- Document new dependencies
- Use virtual environments
- Maintain requirements.txt

# Code Organization
- Keep files focused
- Use meaningful names
- Follow project structure
- Group related code
- Use appropriate abstractions

# Comments
- Write clear comments
- Explain complex logic
- Keep comments up to date
- Use TODO comments sparingly
- Document workarounds

# Configuration
- Use config.json for settings
- Document configuration options
- Use sensible defaults
- Validate configuration
- Keep secrets separate

# Logging
- Use structured logging
- Include context
- Use appropriate levels
- Log important events
- Include timestamps

# Error Messages
- Be specific
- Include context
- Use consistent format
- Provide solutions
- Log stack traces

# API Design
- Use RESTful principles
- Version APIs
- Document endpoints
- Handle errors gracefully
- Use appropriate status codes

# Database
- Use migrations
- Index appropriately
- Handle connections properly
- Use transactions when needed
- Clean up resources

# Caching
- Use appropriate TTLs
- Handle cache misses
- Invalidate properly
- Monitor cache size
- Use cache keys consistently

# Async Code
- Use asyncio properly
- Handle cancellation
- Use timeouts
- Clean up resources
- Handle backpressure

# Testing Best Practices
- Test edge cases
- Use meaningful data
- Clean up after tests
- Mock external services
- Test error conditions

# Documentation Best Practices
- Keep docs up to date
- Include examples
- Document assumptions
- Explain design decisions
- Include troubleshooting

# Code Review
- Review for security
- Check error handling
- Verify logging
- Test edge cases
- Check performance

# Deployment
- Use Docker
- Document process
- Include health checks
- Monitor resources
- Handle rollbacks

# Monitoring
- Use Prometheus metrics
- Monitor queue sizes
- Track error rates
- Monitor performance
- Alert on issues

# Security Best Practices
- Validate input
- Sanitize output
- Use secure defaults
- Handle secrets properly
- Follow OWASP guidelines 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willschneider15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
