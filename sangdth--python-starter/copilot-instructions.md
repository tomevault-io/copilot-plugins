## python-starter

> Context7 queries technical documentation to find current code examples and best practices. Use it to get reliable snippets from official sources.


# Context7 Code Examples Guide

Context7 queries technical documentation to find current code examples and best practices. Use it to get reliable snippets from official sources.

## Two-Step Process

1. **Find Library ID**: Use `resolve-library-id` to get the unique identifier
2. **Get Documentation**: Use `get-library-docs` to fetch examples

## Tools

### `resolve-library-id`

Converts common library names to Context7 IDs.

- **Input**: Library name (e.g., "flask", "django", "requests")
- **Output**: List of matching libraries with their Context7 IDs
- **Use**: Required before fetching documentation

### `get-library-docs`

Retrieves documentation and code examples.

- **Required**: `context7CompatibleLibraryID` (from resolve tool)
- **Optional**: `topic` (narrows results to specific features)

## Example: Flask Hello World

### Step 1: Get Library ID

```
resolve-library-id
Input: "flask"
Output: /pallets/flask
```

### Step 2: Fetch Documentation

```
get-library-docs
Input:
  - context7CompatibleLibraryID: "/pallets/flask"
  - topic: "hello world"
```

**Result**: Returns official Flask documentation with working code:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

## Best Practices

- Use official library names ("requests", not "python-requests")
- Use specific topics ("routing", "authentication", "database models")
- Refine topic searches if results aren't targeted enough
- Repeat process for multiple libraries as needed

## Coordination with DeepWiki

### When to Use Context7 vs DeepWiki:

**Use Context7 when:**

- You need specific code examples from libraries
- You're implementing with a known library/framework
- You need official documentation snippets
- You're looking for implementation patterns

**Use DeepWiki when:**

- You need general research on a topic/concept
- You're exploring new technologies or approaches
- You need AI-grounded answers and clarifications
- You're doing broad research before narrowing down

### Integrated Research Workflow:

1. **DeepWiki Phase**: Use `@deepwiki.mdc` for general research and topic exploration
2. **Context7 Phase**: Use `@context7.mdc` for specific library documentation and code examples
3. **Implementation**: Combine insights from both research phases

### Avoiding Redundancy:

- Use DeepWiki first for broad understanding
- Use Context7 for specific implementation details
- Don't duplicate research phases - coordinate them

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sangdth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
