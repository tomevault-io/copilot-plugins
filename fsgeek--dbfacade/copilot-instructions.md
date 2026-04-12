## dbfacade

> The following files contain critical information about the project:

# CLAUDE.md - DB Facade Development Guidelines

## Key Project Documentation

The following files contain critical information about the project:

1. **[README.md](README.md)** - Project overview and basic usage instructions
2. **[DESIGN.md](DESIGN.md)** - Comprehensive design document with architecture and technical specifications
3. **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines and coding standards
4. **[TODO.md](TODO.md)** - Implementation plan and task list

## CRITICAL: FAIL-STOP IS THE PRIMARY DESIGN PRINCIPLE

DB Facade follows a strict FAIL-STOP model as its primary design principle:

1. NEVER implement fallbacks or paper over errors
2. ALWAYS fail immediately and visibly when issues occur
3. NEVER substitute mock/fake data when real data is unavailable
4. ALWAYS exit with a clear error message (sys.exit(1)) rather than continuing with degraded functionality

This is ESPECIALLY important for database security, where integrity and privacy are critical. Silently substituting mock data when database connections fail would compromise security and is strictly prohibited.

Remember: It is better to fail loudly and immediately than to continue with compromised functionality.

## Available Tools

### MCP ArangoDB Access
Direct database access is available through MCP tools:
- `mcp__arango-mcp__arango_query` - Execute AQL queries directly against the database
- `mcp__arango-mcp__arango_insert` - Insert documents into collections
- `mcp__arango-mcp__arango_update` - Update existing documents
- `mcp__arango-mcp__arango_remove` - Remove documents from collections
- `mcp__arango-mcp__arango_backup` - Create backups of collections
- `mcp__arango-mcp__arango_list_collections` - List all collections in the database
- `mcp__arango-mcp__arango_create_collection` - Create new collections

Use these tools to verify database state, diagnose issues, and confirm that code problems aren't actually database connectivity issues. Following the fail-stop model for database operations, any genuine database connectivity issue requires immediate attention.

## Development Environment

### Package Management
DB Facade uses `uv` for dependency management (defined in `pyproject.toml`):

```bash
# Install uv
pip install uv

# Install dependencies
uv pip install -e .
```

### Virtual Environments
Platform-specific environments:
- Windows: `.venv-win32-python3.12`
- Linux: `.venv-linux-python3.13`
- macOS: `.venv-macos-python3.12`

Always activate before running code:
```bash
# Linux
source .venv-linux-python3.13/bin/activate

# Windows (PowerShell)
.venv-win32-python3.12\Scripts\Activate.ps1

# Windows (CMD)
.venv-win32-python3.12\Scripts\activate.bat

# macOS
source .venv-macos-python3.12/bin/activate
```

### Cross-Platform Development
- All scripts must work with `-help` flag on all platforms
- For Windows-specific code, use conditional imports after CLI parsing
- Don't mask import errors for non-platform-specific packages

## Style Guidelines
- **Imports**: standard library → third-party → local
- **Types**: Use type hints for all functions and variables (Python 3.13+ features encouraged)
- **Formatting**: 4 spaces, ~100 char line length
- **Naming**: CamelCase (classes), snake_case (functions/vars), UPPER_CASE (constants)
- **Interfaces**: Prefix with 'I' (IObject, IRelationship)
- **Documentation**: Docstrings with Args/Returns sections
- **Modern Python**: Use match/case and other Python 3.13+ features where appropriate

### Timezone-Aware Datetime
Always use timezone-aware datetimes for ArangoDB:
```python
from datetime import datetime, timezone
from pydantic import Field

class MyModel(BaseModel):
    # Timezone-aware dates
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))

    @validator('created_at')
    def ensure_timezone(cls, v):
        if v.tzinfo is None:
            return v.replace(tzinfo=timezone.utc)
        return v
```

### Data Models and Pydantic Handling

Always extend BaseModel for database models:
```python
from pydantic import BaseModel

class MyModel(BaseModel):
    name: str
    value: int
```

#### Pydantic V2 Best Practices

DB Facade uses Pydantic V2 for data validation and serialization. Follow these practices:

1. **Never use the deprecated .dict() method**
```python
# INCORRECT - Deprecated in Pydantic V2, will be removed in V3
serialized_data = pydantic_object.dict()  # Will generate warnings

# CORRECT - Use model_dump() for dict conversion
serialized_data = pydantic_object.model_dump()

# CORRECT - For JSON serialization in one step
json_string = pydantic_object.model_dump_json()

# CORRECT - If you need a dictionary from JSON string
doc = json.loads(pydantic_object.model_dump_json())
```

2. **Use model_config instead of Config class**
```python
# INCORRECT - Old style Config class
class OldModel(BaseModel):
    name: str
    
    class Config:
        arbitrary_types_allowed = True

# CORRECT - New style model_config
class NewModel(BaseModel):
    name: str
    
    model_config = {"arbitrary_types_allowed": True}
```

3. **Prefer Field over field for defaults**
```python
from pydantic import BaseModel, Field

class UserModel(BaseModel):
    # Use Field with proper type annotation
    name: str = Field(default="Anonymous")
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
```

## Common Commands

### Testing & Development
- Run tests: `pytest tests/`
- Format code: `black .`
- Lint code: `flake8` or `ruff`

## Error Handling and Fail-Stop Approach

DB Facade uses a strict fail-stop approach for critical errors:

```python
import sys

# CRITICAL ERRORS: Use sys.exit(1) for immediate termination
def critical_operation():
    """Operation that must succeed or the program cannot continue."""
    try:
        result = risky_but_essential_operation()
        if not result:
            logger.error("CRITICAL: Essential operation failed")
            sys.exit(1)  # Exit immediately with error status
        return result
    except Exception as e:
        logger.error(f"CRITICAL: Fatal error in essential operation: {e}")
        sys.exit(1)  # Exit immediately with error status

# NON-CRITICAL ERRORS: Use exceptions for recoverable issues
try:
    result = risky_operation()
except (ValueError, KeyError) as e:
    logger.error(f"Failed to process data: {e}")
    raise DBFacadeProcessingError(f"Data processing failed: {str(e)}") from e
```

#### When to Use Fail-Stop
- For validation errors that would lead to corrupted data
- For essential infrastructure components (database connection, file system)
- When continuing would produce meaningless or incorrect results
- When the same error will occur repeatedly in a loop

#### When Not to Use Fail-Stop
- For recoverable errors where alternate flows exist
- For expected error conditions that can be handled
- In library code called by other components
- During data import where partial success is acceptable

### ArangoDB Cursor Handling
ArangoDB cursors should be handled carefully, especially with large result sets:

```python
# For small result sets, fully consume the cursor
if isinstance(results, Cursor):
    result_list = [doc for doc in results]  # Convert cursor to list
    result_data["results"] = result_list
else:
    result_data["results"] = results
```

For large collections:
```python
# Process cursor in batches with a batch_size parameter
cursor = db.aql.execute(query, bind_vars=params, batch_size=1000)

# Option 1: Cap maximum results to avoid memory issues
results = []
max_results = 10000
for doc in cursor:
    results.append(doc)
    if len(results) >= max_results:
        logging.info(f"Reached maximum result count of {max_results}")
        break

# Option 2: Process results in batches without storing everything
processed = 0
for doc in cursor:
    process_document(doc)  # Process each document individually
    processed += 1
    if processed % 10000 == 0:
        logging.info(f"Processed {processed} documents...")
```

### Logging
```python
import logging

# Start of main function
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fsgeek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fsgeek)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
