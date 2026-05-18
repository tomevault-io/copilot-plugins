## mcp

> Model Context Protocol (MCP) server best practices


# MCP Server Best Practices

Follow these conventions when developing MCP servers.

## Tool Design

### Naming Conventions

Use descriptive, action-oriented names with service prefixes:

```typescript
// Good - clear service prefix and action
"drive_list_files"
"docs_create"
"calendar_get_event"
"gmail_send_message"

// Bad - unclear or inconsistent
"listFiles"
"getDoc"
"sendEmail"
```

### Input Schemas

Always provide comprehensive JSON schemas for tool inputs:

```typescript
{
  name: "calendar_create_event",
  description: "Create a new calendar event with title, time, and optional attendees",
  inputSchema: {
    type: "object",
    properties: {
      title: {
        type: "string",
        description: "Event title/summary"
      },
      startTime: {
        type: "string",
        description: "Start time in ISO 8601 format (e.g., 2024-01-15T10:00:00Z)"
      },
      endTime: {
        type: "string",
        description: "End time in ISO 8601 format"
      },
      attendees: {
        type: "array",
        items: { type: "string" },
        description: "Email addresses of attendees (optional)"
      }
    },
    required: ["title", "startTime", "endTime"]
  }
}
```

### Descriptions

Write clear, actionable descriptions that help the LLM understand:
- What the tool does
- When to use it
- What parameters mean
- What the output contains

```typescript
// Good
description: "Search for files in Google Drive by name, type, or content. Returns file IDs, names, and metadata. Use this to find files before performing operations on them."

// Bad
description: "Search files"
```

## Error Handling

### Return Structured Errors

Always return errors in a consistent format:

```typescript
if (!this.isAuthenticated()) {
  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        error: "NotAuthenticated",
        message: "Please authenticate first using the google_auth tool",
        suggestion: "Run the google_auth tool to start the OAuth flow"
      }, null, 2)
    }],
    isError: true
  };
}
```

### Validate Inputs Early

Check required parameters before calling external APIs:

```typescript
public async handleTool(name: string, args: Record<string, unknown>): Promise<ToolResponse> {
  if (name === "docs_create") {
    const { title } = args as { title?: string };

    if (!title?.trim()) {
      return {
        content: [{ type: "text", text: "Error: title is required and cannot be empty" }],
        isError: true
      };
    }

    // Proceed with API call
  }
}
```

## Resource Design

### URI Patterns

Use consistent, hierarchical URI patterns:

```typescript
// Good patterns
"drive://files"
"drive://files/{fileId}"
"docs://documents/{docId}"
"calendar://events?date=2024-01-15"

// Bad patterns
"getFiles"
"document-123"
```

### Resource Metadata

Provide helpful metadata for resources:

```typescript
{
  uri: "drive://files/abc123",
  name: "Project Proposal.docx",
  description: "Google Docs document last modified 2024-01-15",
  mimeType: "application/vnd.google-apps.document"
}
```

## Authentication

### Handle Auth State Gracefully

Always check authentication before operations:

```typescript
private ensureAuthenticated(): void {
  if (!oauth.isReady()) {
    throw new Error(
      "Not authenticated. Please run the google_auth tool first to authenticate with Google."
    );
  }
}

public async handleTool(name: string, args: unknown): Promise<ToolResponse> {
  try {
    this.ensureAuthenticated();
    // ... tool logic
  } catch (error) {
    return {
      content: [{ type: "text", text: `Error: ${error.message}` }],
      isError: true
    };
  }
}
```

### Provide Auth Tools

Include authentication management tools:

```typescript
// Essential auth tools
"google_auth"        // Start OAuth flow
"google_auth_status" // Check current auth state
"google_auth_code"   // Manual code entry fallback
"google_logout"      // Revoke access
```

## Response Formatting

### Return Structured Data

Return JSON for complex data, formatted for readability:

```typescript
return {
  content: [{
    type: "text",
    text: JSON.stringify(result, null, 2)
  }]
};
```

### Include Helpful Context

Add context that helps the LLM understand results:

```typescript
const result = {
  success: true,
  fileId: "abc123",
  fileName: "Document.docx",
  webViewLink: "https://docs.google.com/...",
  message: "Document created successfully. Use the webViewLink to open it in a browser."
};
```

## Service Architecture

### Separate Concerns

Organize code into focused service classes:

```
src/
├── server.ts           # MCP server, tool routing
├── auth/
│   └── oauth.ts        # Authentication logic
├── services/
│   ├── drive.ts        # Google Drive operations
│   ├── docs.ts         # Google Docs operations
│   └── ...
└── types/
    └── index.ts        # Shared types and schemas
```

### Initialize Services Lazily

Only initialize services when authenticated:

```typescript
private initializeServices(): void {
  const client = oauth.getClient();
  if (client) {
    this.drive = new DriveService(client);
    this.docs = new DocsService(client);
    // ...
  }
}
```

## Performance

### Use Pagination

Support pagination for list operations:

```typescript
{
  name: "drive_list_files",
  inputSchema: {
    properties: {
      pageSize: {
        type: "number",
        description: "Maximum files to return (default: 10, max: 100)"
      },
      pageToken: {
        type: "string",
        description: "Token for next page of results"
      }
    }
  }
}
```

### Return Relevant Fields Only

Don't return entire API responses; extract useful fields:

```typescript
// Good - curated response
return {
  id: file.id,
  name: file.name,
  mimeType: file.mimeType,
  webViewLink: file.webViewLink
};

// Bad - raw API dump
return response.data;
```

## Testing Tools

When developing, test tools with various inputs:

1. Valid inputs with all parameters
2. Valid inputs with only required parameters
3. Invalid/missing required parameters
4. Edge cases (empty strings, special characters)
5. Unauthenticated state

---
> Source: [quinnjr/google-mcp](https://github.com/quinnjr/google-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
