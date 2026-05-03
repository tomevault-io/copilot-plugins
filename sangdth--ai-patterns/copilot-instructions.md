## context7

> Get reliable snippets from official sources

# Context7 Code Examples Guide

Context7 queries technical documentation to find current code examples and
best practices.

## Two-Step Process

1. **Find Library ID**: Use `resolve-library-id` to get the unique identifier
2. **Get Documentation**: Use `get-library-docs` to fetch examples

## Tools

### `resolve-library-id`

Converts common library names to Context7 IDs.

- **Input**: Library name (e.g., "express", "react")
- **Output**: List of matching libraries with their Context7 IDs
- **Use**: Required before fetching documentation

### `get-library-docs`

Retrieves documentation and code examples.

- **Required**: `context7CompatibleLibraryID` (from resolve tool)
- **Optional**: `topic` (narrows results to specific features)

## Example: Express Hello World

### Step 1: Get Library ID

```txt
resolve-library-id
Input: "express"
Output: /expressjs/express
```

### Step 2: Fetch Documentation

```txt
get-library-docs
Input:
  - context7CompatibleLibraryID: "/expressjs/express"
  - topic: "hello world"
```

**Result**: Returns official Express.js documentation with working code:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.info(`Example app listening on port ${port}`);
});
```

## Best Practices

- Use official library names ("react", not "react.js")
- Use specific topics ("middleware", "routing", "authentication")
- Refine topic searches if results aren't targeted enough
- Repeat process for multiple libraries as needed

## Coordination with DeepWiki

### When to Use Context7 vs DeepWiki

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

### Integrated Research Workflow

1. **DeepWiki Phase**: Use `@deepwiki.mdc` for general research and topic
   exploration
2. **Context7 Phase**: Use `@context7.mdc` for specific library documentation
   and code examples
3. **Implementation**: Combine insights from both research phases

### Avoiding Redundancy

- Use DeepWiki first for broad understanding
- Use Context7 for specific implementation details
- Don't duplicate research phases - coordinate them

---
> Source: [sangdth/ai-patterns](https://github.com/sangdth/ai-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
