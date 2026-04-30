## ai-assisted-workflow

> Code style guidelines and conventions


# Code Style Guidelines

## General Principles

### Consistency Over Preference

- Follow existing patterns in the codebase
- Consistency within a file/module is paramount
- When in doubt, match surrounding code style

### Readability Over Cleverness

- Write code for humans first, machines second
- Prefer explicit over implicit
- Use descriptive names over short abbreviations

### Simplicity Over Optimization

- Clear code beats clever code
- Optimize only when necessary (after profiling)
- Document why when breaking this rule

## File Organization

### Directory Structure

```
project-root/
├── .agents/               # Centralized agent configurations
│   ├── mcp/              # MCP server configs (source of truth)
│   ├── rules/            # Project rules (source of truth)
│   └── skills/           # Agent skills (source of truth)
├── .cursor/              # Cursor-specific (symlinks/generated)
├── .claude/              # Claude Code-specific (symlinks/generated)
├── .gemini/              # Gemini CLI-specific (symlinks/generated)
├── .agents/workflows     # Antigravity workflows (symlink → commands)
├── docs/                 # Documentation
│   ├── guidelines/       # Project-specific coding standards
│   ├── guides/           # How-to guides
│   ├── notes/            # Research and comparisons
│   └── references/       # Technical documentation
└── src/                  # Source code (if applicable)
```

### File Naming

**Markdown Files:**

- Use kebab-case: `code-style.md`, `mcp-setup-guide.md`
- Be descriptive: `antigravity-limitation.md` not `limit.md`
- Use `.md` extension for all markdown

**Scripts:**

- Use kebab-case: `sync.sh`, `protect-secrets.sh`
- Include `.sh` extension for bash scripts
- Make executable: `chmod +x script.sh`

**Config Files:**

- Use platform conventions:
  - `mcp.json` (Cursor/Claude)
  - `settings.json` (Gemini)
  - `mcp_config.json` (Antigravity)

## Bash Scripts

### Script Structure

```bash
#!/bin/bash

set -e  # Exit on error

# Constants at top
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"

# Parse arguments
DRY_RUN=false
if [[ "$1" == "--dry-run" ]]; then
  DRY_RUN=true
fi

# Functions before main
validate_sources() {
  # ...
}

# Main execution at bottom
main() {
  validate_sources
  # ...
}

main
```

### Bash Best Practices

**Error Handling:**

```bash
# Exit on error
set -e

# Check command exists
if ! command -v jq &> /dev/null; then
  echo "❌ jq not installed"
  exit 1
fi

# Check file exists
if [ ! -f "$CONFIG_FILE" ]; then
  echo "❌ Config not found: $CONFIG_FILE"
  exit 1
fi
```

**Path Resolution:**

```bash
# Get script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Get project root (relative to script)
PROJECT_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"

# Use absolute paths
CONFIG="$PROJECT_ROOT/.agents/mcp/mcp-servers.json"
```

**Output Formatting:**

```bash
# Use emoji for visual clarity
echo "🔄 Processing..."
echo "  ✅ Success"
echo "  ⚠️  Warning"
echo "  ❌ Error"

# Indentation for hierarchy
echo "Main task:"
echo "  Subtask 1"
echo "    Detail"
```

**Dry Run Support:**

```bash
DRY_RUN=false

if [ "$DRY_RUN" = true ]; then
  echo "  [DRY RUN] Would create file: $FILE"
  return 0
fi

# Actual operation
touch "$FILE"
```

## Markdown Documentation

### Document Structure

```markdown
# Title (H1)

Brief introduction paragraph.

## Section (H2)

### Subsection (H3)

Content with examples.

#### Sub-subsection (H4)

Use sparingly for deep nesting.
```

### Code Blocks

**Always specify language:**

````markdown
```bash
echo "Good example"
```
````

```json
{
  "key": "value"
}
```

````

**Use syntax highlighting:**
- `bash` for shell commands
- `json` for JSON files
- `markdown` for markdown examples
- `yaml` for YAML files

### Links

**Relative paths for internal docs:**
```markdown
See [MCP Setup Guide](docs/guides/mcp/mcp-setup-guide.md)
```

**Absolute URLs for external:**
```markdown
[Context7 Dashboard](https://context7.com/dashboard)
```

### Lists

**Unordered lists for non-sequential items:**
```markdown
- First item
- Second item
  - Nested item
  - Another nested
```

**Ordered lists for sequential steps:**
```markdown
1. First step
2. Second step
3. Third step
```

**Definition lists for key-value pairs:**
```markdown
**Term:** Definition here
**Another term:** Another definition
```

## JSON Configuration

### Formatting

```json
{
  "key": "value",
  "array": [
    "item1",
    "item2"
  ],
  "nested": {
    "deep": "value"
  }
}
```

**Rules:**
- 2-space indentation
- No trailing commas
- Double quotes for strings
- Newline at end of file

### MCP Server Configuration

```json
{
  "servers": {
    "server-name": {
      "platforms": ["cursor", "claude", "gemini"],
      "description": "Server description",
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "${VARIABLE_NAME}"
      }
    }
  }
}
```

## Comments and Documentation

### When to Comment

**DO comment:**
- Complex logic or algorithms
- Non-obvious workarounds
- Platform-specific limitations
- Security considerations

**DON'T comment:**
- Obvious code (`i++` // increment i)
- Redundant explanations
- Commented-out code (delete it)

### Comment Style

**Bash:**
```bash
# Single line comment

# Multi-line explanation
# continues on next line
# and another line
```

**Markdown:**
```markdown
<!-- HTML comments for notes not shown to users -->
```

## Naming Conventions

### Variables

**Bash:**
```bash
# Constants: UPPER_SNAKE_CASE
PROJECT_ROOT="/path/to/project"
CONFIG_FILE="config.json"

# Variables: lower_snake_case
dry_run=false
config_path="/path"
```

**JSON:**
```json
{
  "camelCase": "for keys",
  "nested": {
    "alsoConsistent": true
  }
}
```

### Functions

**Bash:**
```bash
# Descriptive verb_noun format
validate_sources() {
  # ...
}

create_directory_symlink() {
  # ...
}

sync_cursor() {
  # ...
}
```

## Git Conventions

### Commit Messages

```
type: Brief description (50 chars or less)

Detailed explanation if needed. Wrap at 72 characters.

- Bullet points for multiple changes
- Each change on its own line

Refs: #123
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation only
- `refactor:` Code restructuring
- `test:` Adding tests
- `chore:` Maintenance tasks

### Branch Naming

```
feature/short-description
fix/bug-description
docs/documentation-update
refactor/code-restructure
```

## Security

### Never Commit Secrets

**Use environment variables:**
```bash
# Good
export CONTEXT7_API_KEY="your-api-key"

# Bad - never commit
API_KEY="sk-abc123"
```

**Use placeholder values in examples:**
```json
{
  "env": {
    "API_KEY": "${CONTEXT7_API_KEY}"
  }
}
```

### File Permissions

**Scripts should be executable:**
```bash
chmod +x script.sh
```

**Configs should not be executable:**
```bash
chmod 644 config.json
```

## Platform-Specific Considerations

### Antigravity Limitations

```bash
# Antigravity doesn't support project-level MCP configs
# Must copy rules instead of symlink
if [ "$PLATFORM" = "antigravity" ]; then
  cp -r "$SOURCE" "$DEST"
else
  ln -s "$SOURCE" "$DEST"
fi
```

### Path Compatibility

```bash
# Use forward slashes (works on macOS/Linux/WSL)
PATH="/Users/user/project/file.txt"

# Avoid backslashes (Windows-specific)
# PATH="C:\Users\user\project\file.txt"  # Don't do this
```

## References

- [Bash Best Practices](https://bertvv.github.io/cheat-sheets/Bash.html)
- [Markdown Guide](https://www.markdownguide.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- `.agents/sync.sh` - Unified sync CLI (reference implementation)
````

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LIDR-IT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
