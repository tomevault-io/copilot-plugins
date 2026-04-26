## monday-cli

> This is a CLI tool for interacting with the Monday.com GraphQL API.

# Claude Instructions for Monday CLI

This is a CLI tool for interacting with the Monday.com GraphQL API.

## Project Structure

```
src/monday_cli/
├── __init__.py          # Package init with version
├── cli.py               # Main CLI entry point with all command group registrations
├── client/
│   ├── client.py        # HTTP client with retry/rate limiting
│   ├── models.py        # Pydantic models for API responses
│   ├── queries.py       # GraphQL query templates
│   └── mutations.py     # GraphQL mutation templates
├── commands/
│   ├── workspaces.py    # Workspace management commands
│   ├── boards.py        # Board management commands
│   ├── groups.py        # Group management commands
│   ├── items.py         # Item management commands
│   ├── subitems.py      # Subitem management commands
│   ├── statuses.py      # Status listing commands
│   ├── updates.py       # Update management commands
│   └── docs.py          # Document management commands
└── utils/
    ├── output.py        # Output formatting utilities (JSON, tables)
    └── error_handler.py # Error handling and custom exceptions
```

## Monday.com API

### Authentication

Get your API token from: https://apphammer.monday.com/admin/integrations/api

Set the token as environment variable:
```bash
export MONDAY_API_TOKEN="your_token_here"
```

### API Endpoint

- Base URL: `https://api.monday.com/v2`
- All requests are POST with GraphQL body
- Authentication via `Authorization: your_api_token` header

### GraphQL Patterns

#### Queries

Items query requires `ID!` type (string, not int):
```graphql
query GetItem($itemIds: [ID!]!) {
  items(ids: $itemIds) {
    id
    name
    board { id name }
    column_values { id text value type }
  }
}
```

Board columns query (includes status settings):
```graphql
query GetBoardColumns($boardIds: [ID!]!) {
  boards(ids: $boardIds) {
    columns {
      id
      title
      type
      settings_str  # JSON string with status labels
    }
  }
}
```

#### Mutations

Create item:
```graphql
mutation CreateItem($boardId: ID!, $itemName: String!, $columnValues: JSON) {
  create_item(board_id: $boardId, item_name: $itemName, column_values: $columnValues) {
    id
    name
  }
}
```

Update column value (for status changes):
```graphql
mutation ChangeColumnValue($boardId: ID!, $itemId: ID!, $columnId: String!, $value: JSON!) {
  change_column_value(board_id: $boardId, item_id: $itemId, column_id: $columnId, value: $value) {
    id
  }
}
```

### Status Columns

Status columns have `settings_str` containing JSON with label mappings:
```json
{
  "labels": {
    "0": "Done",
    "1": "Working on it",
    "2": "Stuck"
  }
}
```

To update a status, use the index in the value:
```json
{"index": 1}  // Sets status to "Working on it"
```

### Rate Limiting

- Monday.com has complexity-based rate limiting
- Track complexity via `complexity { before after }` in queries
- CLI implements 60 calls/minute conservative limit
- Retry logic with exponential backoff (1s, 2s, 4s)

### Common Column Types

| Type | Description | Value Format |
|------|-------------|--------------|
| `status` | Status dropdown | `{"index": N}` |
| `text` | Plain text | `"string value"` |
| `date` | Date picker | `"YYYY-MM-DD"` |
| `numbers` | Numeric | `"123"` or `123` |
| `people` | Person picker | `{"personsAndTeams": [{"id": N, "kind": "person"}]}` |
| `dropdown` | Dropdown | `{"ids": [1, 2]}` |

## Development

### Build Binary

```bash
python build/build_binary.py
```

Creates standalone Linux binary at `dist/monday`.

### Run from Source

```bash
python -m monday_cli --help
```

### Testing

```bash
pytest
```

## CLI Framework

Uses Typer (v0.21.0+) with **8 command groups**:

### Command Groups Overview

1. **`monday workspaces <command>`** - Workspace operations
   - `list` - List workspaces with membership filtering

2. **`monday boards <command>`** - Board operations
   - `list` - List boards with state and workspace filtering

3. **`monday groups <command>`** - Group operations
   - `list` - List groups on a board
   - `create` - Create new group
   - `delete` - Delete group

4. **`monday items <command>`** - Item operations
   - `get` - Get item details
   - `list` - List items with pagination and group filtering
   - `create` - Create new item
   - `update` - Update item column value using human-readable titles
   - `delete` - Delete item with confirmation
   - `list-columns` - List all board columns with types and options

5. **`monday subitems <command>`** - Subitem operations
   - `get` - Get subitem details
   - `list` - List subitems with pagination (by item or board)
   - `create` - Create new subitem
   - `update` - Update subitem column value using human-readable titles
   - `delete` - Delete subitem with confirmation
   - `list-columns` - List all board columns
   - `list-statuses` - List status columns with options

6. **`monday statuses <command>`** - Status operations
   - `list` - List all status columns and options for a board

7. **`monday updates <command>`** - Update operations
   - `create` - Post update to item or subitem

8. **`monday docs <command>`** - Document operations
   - `get` - Get document content as Markdown
   - `append` - Append Markdown content to document (creates if needed)
   - `put` - Replace document content with Markdown (clears existing content first)

### Built-in Commands
- `monday version` - Show version information
- `monday --help` - Show help
- `monday <group> --help` - Show group-specific help

## Coding Guidelines

### Adding New Commands

1. **Choose the right file**: Add to existing command files based on the resource type:
   - `commands/workspaces.py` - Workspace operations
   - `commands/boards.py` - Board operations
   - `commands/groups.py` - Group operations
   - `commands/items.py` - Item operations
   - `commands/subitems.py` - Subitem operations
   - `commands/statuses.py` - Status listing operations
   - `commands/updates.py` - Update operations
   - `commands/docs.py` - Document operations
   - Create a new file only for entirely new resource types

2. **Command structure template**:
```python
@items_app.command("command-name")
def command_name(
    required_arg: int = typer.Argument(..., help="Description"),
    optional_arg: Optional[str] = typer.Option(None, "--flag", "-f", help="Description"),
) -> None:
    """Short description of what the command does.

    Example:
        monday items command-name 1234567890
    """
    try:
        client = get_client()
        # ... implementation ...
        print_json(result)

    except AuthenticationError:
        typer.secho(
            "Error: Invalid API token. Set MONDAY_API_TOKEN environment variable.",
            fg=typer.colors.RED,
        )
        raise typer.Exit(1)
    except RateLimitError as e:
        typer.secho(f"Error: {str(e)}", fg=typer.colors.YELLOW)
        raise typer.Exit(1)
    except MondayAPIError as e:
        typer.secho(f"API Error: {str(e)}", fg=typer.colors.RED)
        raise typer.Exit(1)
    except Exception as e:
        typer.secho(f"Unexpected error: {str(e)}", fg=typer.colors.RED)
        raise typer.Exit(1)
```

3. **Required imports for commands**:
```python
import json
from typing import Optional

import typer

from monday_cli.cli import get_client, items_app  # or workspaces_app, boards_app, groups_app, subitems_app, statuses_app, updates_app, docs_app
from monday_cli.client.mutations import CHANGE_COLUMN_VALUE, CREATE_ITEM
from monday_cli.client.queries import GET_BOARD_COLUMNS, GET_ITEM_BY_ID
from monday_cli.utils.error_handler import AuthenticationError, MondayAPIError, RateLimitError
from monday_cli.utils.output import print_json, print_table  # Use print_table for --table option
```

### Adding New GraphQL Operations

1. **Queries** go in `client/queries.py`:
```python
GET_SOMETHING = """
query GetSomething($ids: [ID!]!) {
  something(ids: $ids) {
    id
    name
  }
  complexity {
    before
    after
  }
}
"""
```

2. **Mutations** go in `client/mutations.py`:
```python
DO_SOMETHING = """
mutation DoSomething($id: ID!, $value: String!) {
  do_something(id: $id, value: $value) {
    id
  }
}
"""
```

3. **Always include complexity tracking** in queries for rate limit monitoring.

### ID Type Conventions

- **Monday.com IDs** are strings in GraphQL but often passed as integers from CLI
- Always convert to string when sending to API: `str(item_id)`
- Use `int` type hint for CLI arguments for better validation
- GraphQL uses `ID!` type which accepts strings

### Output Conventions

- Use `print_json()` for all data output (enables machine-readable output)
- Use `typer.secho()` for user messages:
  - Success: `fg=typer.colors.GREEN`
  - Warning/not found: `fg=typer.colors.YELLOW`
  - Error: `fg=typer.colors.RED`
- Use checkmark for success: `"✓ Action completed successfully!"`

### Error Handling Pattern

Always handle these exceptions in order:
1. `AuthenticationError` - Invalid API token
2. `RateLimitError` - Rate limit exceeded
3. `MondayAPIError` - API-specific errors
4. `Exception` - Catch-all for unexpected errors

Always `raise typer.Exit(1)` after error messages.

### Development Guidelines

- always use named arguments over positional for clarity
  `monday items get --item-id 123` instead of `monday items get 123`
- all `list` commands should support `--table` option for tabular output
- `list` commands with pagination should support:
  - `--limit` (1-500) for page size
  - `--cursor` for continuation
  - `--all` for automatic pagination through all results
- standard commands `list`, `get`, `create`, `update`, `delete` for resources

## Key Features Reference

### Pagination Support

Commands that support cursor-based pagination:
- `monday items list` - Paginate through board items
- `monday subitems list` (when using `--board-id`) - Paginate through subitems
- `monday boards list` - Automatic pagination built-in (max 100 per page)

Pagination options:
```bash
--limit INT    # Items per page (1-500, default: 100)
--cursor TEXT  # Pagination cursor for next page
--all          # Fetch all items across all pages automatically
```

The API returns pagination info:
```json
{
  "items": [...],
  "pagination": {
    "cursor": "MSw5NzI4MDA5MDAsaV9YcmxJb0p1VEdYc1VWeGlxeF9kLDg4MiwzNXw0MTQ1NzU1MTE5",
    "has_more": true,
    "total_items": 150,
    "pages_fetched": 1
  }
}
```

### Column Value Updates

The `items update` and `subitems update` commands support intelligent column type detection:

**Supported column types:**
- **status**: Matches label case-insensitively, converts to `{"index": N}`
- **text**: Direct string value
- **link**: URL string, converts to `{"url": "value", "text": "value"}`
- **date**: Date string (YYYY-MM-DD), converts to `{"date": "value"}`
- **numbers**: Numeric string or number
- **long-text**: Text content (for long text columns)

**Auto-detection workflow:**
1. Look up column by title (case-insensitive)
2. Detect column type from board schema
3. Format value appropriately for the type
4. For status: match label and find index, show available options if not found
5. Update using the Monday.com API

Example:
```bash
monday items update --item-id 123 --title "Status" --value "done"
# Auto-detects status column, finds "Done" label, converts to {"index": 0}
```

### Table Output

All `list` commands support `--table` flag for rich table formatting:
- Formatted columns with proper alignment
- Color-coded headers
- Human-readable layout
- Supports: workspaces, boards, groups, items, subitems, statuses

Example:
```bash
monday items list --board-id 123 --table
```

### Filtering and Querying

**Workspace filtering:**
- `--membership-kind` (all, member)
- `--workspace-ids` (comma-separated IDs)

**Board filtering:**
- `--state` (active, archived, deleted, all)
- `--workspace-name` (case-insensitive name matching)
- `--workspace-id` (exact ID match)

**Item filtering:**
- `--group` (case-insensitive group title)
- `--group-id` (exact group ID)

**Examples:**
```bash
# Active boards in a workspace
monday boards list --state active --workspace-name "Marketing"

# Items from specific group
monday items list --board-id 123 --group "Sprint 1"
```

### Group Management

Groups organize items on boards and support:
- Color customization (hex codes)
- Deletion confirmation (with `--confirm` flag to skip)
- Position tracking

Color format: `#ff642e` or `#f09`

### Document Operations

Monday.com doc columns can be managed programmatically:
- Create documents with optional initial content
- Replace document content (`put`) - clears all existing blocks, then writes new markdown
- Append to document content (`append`) - adds markdown content without clearing
- Retrieve document content as Markdown (`get`)
- Lookup columns by name (case-insensitive)
- Validates column type is `doc`

The `docs get` command returns Markdown content (falls back to block JSON if export not supported).

### Testing Changes

```bash
# Run from source
python -m monday_cli --help
python -m monday_cli items --help

# Build and test binary
python build/build_binary.py
./dist/monday --help
```

### Style Guidelines

- Use kebab-case for command names: `list-columns`, `update-status`
- Use snake_case for function names: `list_columns`, `update_status`
- Include docstrings with examples for all commands
- Keep command help text concise but descriptive

## Common Usage Examples

### Workflow: Creating and Managing Items

```bash
# 1. Find your workspace and board
monday workspaces list --table
monday boards list --workspace-name "Engineering" --table

# 2. Create a group for organization
monday groups create --title "Sprint 5" --board-id 1234567890 --color "#ff642e"

# 3. Create an item
monday items create --board-id 1234567890 --name "Build new feature" --group-id "sprint_5"

# 4. Update item status
monday items update --item-id 9876543210 --title "Status" --value "Working on it"

# 5. Add update/comment
monday updates create --item-id 9876543210 --body "Started implementation"

# 6. Create subitems for task breakdown
monday subitems create --parent-item-id 9876543210 --name "Write tests"
monday subitems create --parent-item-id 9876543210 --name "Code review"

# 7. Update subitem status
monday subitems update --subitem-id 1111111111 --title "Status" --value "Done"

# 8. List all items with filtering
monday items list --board-id 1234567890 --group "Sprint 5" --all --table
```

### Workflow: Discovering Board Structure

```bash
# 1. List all columns on a board
monday items list-columns --item-id 9876543210

# 2. See available status options
monday statuses list --board-id 1234567890 --table

# 3. List groups on a board
monday groups list --board-id 1234567890 --table

# 4. List items by group
monday items list --board-id 1234567890 --group-id "topics" --table
```

### Workflow: Document Management

```bash
# Replace document content with Markdown (clears existing, then writes new)
monday docs put --item-id 9876543210 --column-name "Notes" --content "# Project Requirements\n\n- Item 1\n- Item 2"

# Append content to existing document (or create new)
monday docs append --item-id 9876543210 --column-name "Notes" --content "## Additional Notes\n\n- Item 3"

# Read document content as Markdown
monday docs get --item-id 9876543210 --column-name "Notes"
```

### Workflow: Deleting Items and Subitems

```bash
# Delete an item (with confirmation prompt)
monday items delete --item-id 9876543210

# Delete an item without confirmation (for scripts)
monday items delete --item-id 9876543210 --force

# Delete a subitem (with confirmation prompt)
monday subitems delete --subitem-id 1111111111

# Delete a subitem without confirmation (for scripts)
monday subitems delete --subitem-id 1111111111 --force
```

### Workflow: Pagination

```bash
# List first page of items
monday items list --board-id 1234567890 --limit 50

# Continue from cursor (copy cursor from previous response)
monday items list --board-id 1234567890 --cursor "MSw5NzI4MDA5MDAsaV9YcmxJb..."

# Or fetch all automatically
monday items list --board-id 1234567890 --all
```

## API Reference

- [Monday.com API Docs](https://developer.monday.com/api-reference/reference/about-the-api-reference)
- [GraphQL Guide](https://developer.monday.com/api-reference/docs/introduction-to-graphql)
- [Column Types](https://developer.monday.com/api-reference/docs/column-types-reference)
- [Rate Limits](https://developer.monday.com/api-reference/docs/rate-limits)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AppHammer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
