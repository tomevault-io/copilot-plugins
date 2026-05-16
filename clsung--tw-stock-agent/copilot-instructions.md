## tw-stock-agent

> **tw-stock-agent** is a Taiwan stock market data MCP (Model Context Protocol) server providing comprehensive financial data analysis. The project consists of:

# tw-stock-agent Development Guidelines

## Project Overview
**tw-stock-agent** is a Taiwan stock market data MCP (Model Context Protocol) server providing comprehensive financial data analysis. The project consists of:

- **MCP Server**: FastMCP-based server with stock data tools
- **Stock Services**: Core business logic for data retrieval and analysis  
- **Data Scripts**: Standalone Taiwan financial data collectors
- **API Layer**: FastAPI endpoints for tool execution

## Core Development Rules

### 1. Package Management
- **ONLY use uv**, NEVER pip
- Installation: `uv add package`
- Development packages: `uv add --dev package`
- Running tools: `uv run tool`
- Sync dependencies: `uv sync`
- **FORBIDDEN**: `uv pip install`, `@latest` syntax

### 2. Code Quality Standards
- **Type hints required** for all functions and methods
- **Docstrings mandatory** for public APIs (Google style)
- Functions must be focused and small (single responsibility)
- Follow existing patterns exactly
- **Line length**: 88 characters maximum (configured in pyproject.toml)
- Use f-strings for string formatting

### 3. Testing Framework
- **Primary**: `uv run pytest`
- **Async testing**: pytest-asyncio (configured in pyproject.toml)
- **Coverage**: `uv run pytest --cov=tw_stock_agent`
- New features require comprehensive tests
- Bug fixes require regression tests
- Test both unit and integration scenarios

### 4. Code Style Guidelines
- **Naming**: PEP 8 conventions
  - Functions/variables: `snake_case`
  - Classes: `PascalCase`  
  - Constants: `UPPER_SNAKE_CASE`
- **Import organization**: Use ruff's isort integration
- **Error handling**: Comprehensive exception handling with logging
- **Logging**: Use structured logging with appropriate levels

### 5. Quality Tools
- **Linting**: `uv run ruff check` (configured in pyproject.toml)
- **Formatting**: `uv run ruff format`
- **Type checking**: `uv run mypy tw_stock_agent/`
- **Pre-commit**: Configured with ruff, mypy, and standard hooks

## Project Structure

```
tw_stock_agent/
├── mcp_server.py           # MCP server entry point (FastMCP)
├── main.py                 # FastAPI application
├── services/
│   ├── stock_service.py    # Core stock data service
│   └── cache_service.py    # Caching implementation
├── tools/
│   ├── stock_tools.py      # MCP tool implementations
│   ├── stock_code.py       # Stock code utilities
│   └── *.csv              # Stock exchange data
├── utils/
│   ├── config.py          # Configuration management
│   ├── data_fetcher.py    # Data fetching utilities
│   ├── error_handler.py   # Error handling
│   └── rate_limiter.py    # API rate limiting
└── db/                    # Database modules
```

## Development Workflow

### Setup
```bash
# Clone and setup
git clone <repository>
cd tw-stock-agent
uv sync

# Install pre-commit hooks
uv run pre-commit install
```

### Quality Checks
```bash
# Run all quality checks
uv run ruff check --fix       # Lint and auto-fix
uv run ruff format           # Format code
uv run mypy tw_stock_agent/  # Type checking
uv run pytest --cov         # Run tests with coverage
```

### MCP Server Development
```bash
# Run MCP server
uv run python mcp_server.py

# Run FastAPI server  
uv run uvicorn tw_stock_agent.main:app --reload
```

### Data Scripts
```bash
# Taiwan financial data collection
uv run python scripts/download_twse.py
uv run python scripts/download_tpex.py
uv run python scripts/download_fsc.py
uv run python scripts/download_cbc.py
```

## Configuration Management

### Environment Variables
- `LOG_LEVEL`: Logging level (DEBUG, INFO, WARNING, ERROR)
- `API_HOST`: FastAPI host (default: 127.0.0.1)
- `API_PORT`: FastAPI port (default: 8000)
- Additional settings in `tw_stock_agent/utils/config.py`

### Settings Priority
1. Environment variables
2. Configuration files  
3. Default values

## Stock Market Data Sources

### Supported Exchanges
- **TWSE** (Taiwan Stock Exchange): Main board stocks, TAIEX index
- **TPEx** (Taipei Exchange): OTC stocks, emerging market

### Data Types
- Stock basic information (company profiles, industry sectors)
- Historical price data (OHLCV)
- Real-time data (current prices, volumes)
- Technical analysis (Best Four Points)
- Market overview (indices, market statistics)

### Rate Limiting
- TWSE API: 3 requests per 5 seconds
- Automatic retry with exponential backoff
- Configurable timeouts and retry limits

## Error Handling

### Exception Strategy
- Use custom exceptions in `utils/error_handler.py`
- Log all errors with appropriate context
- Graceful degradation for non-critical failures
- Return meaningful error messages to clients

### Logging Standards
```python
import logging
logger = logging.getLogger(__name__)

# Error with context
logger.error("Stock data fetch failed", extra={
    "stock_code": stock_code,
    "error": str(e),
    "retry_count": retry_count
})
```

## MCP Integration

### Tool Development
- Implement tools in `tools/stock_tools.py`
- Register tools in both `mcp_server.py` and `main.py`
- Follow MCP tool schema specifications
- Include comprehensive input validation

### Resource Patterns
```python
# MCP resource URI patterns
stock://info/{stock_code}      # Basic stock information
stock://price/{stock_code}     # Historical prices  
stock://realtime/{stock_code}  # Real-time data
```

## Performance Guidelines

### Caching Strategy
- Real-time data: 1-minute cache
- Historical data: 30-minute cache
- Use `cachetools` for in-memory caching
- Implement cache invalidation strategies

### Async Programming
- Use async/await for I/O operations
- Configure proper connection pooling
- Handle concurrent requests efficiently

## Security Considerations

### Data Access
- Public market data only (no authentication required)
- Validate all input parameters
- Sanitize stock codes and date ranges
- Rate limiting to prevent abuse

### Error Information
- Don't expose internal system details
- Log sensitive information separately
- Use structured error responses

## Deployment

### Production Checklist
- Set appropriate log levels
- Configure rate limiting
- Set up monitoring and alerting
- Use environment-specific configuration
- Run quality checks in CI/CD

### Docker Support
- Container-ready architecture
- Minimal resource requirements
- Health check endpoints
- Configurable through environment variables

## Contributing

### Pull Request Process
1. Run all quality checks locally
2. Ensure tests pass with coverage > 80%
3. Update documentation if needed
4. Follow conventional commit messages
5. Request review from maintainers

### Code Review Criteria
- Code quality and style compliance
- Test coverage and quality
- Performance implications
- Security considerations
- Documentation completeness

---
> Source: [clsung/tw-stock-agent](https://github.com/clsung/tw-stock-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
