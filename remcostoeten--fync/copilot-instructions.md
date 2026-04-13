## fync

> description: Fync API integration architecture and style guide for consistent implementation patterns

---
description: Fync API integration architecture and style guide for consistent implementation patterns
globs: ["src/**/*.ts", "fync/src/**/*.ts"]
alwaysApply: true
---

# Fync Architecture Style Guide

This rule defines the EXACT patterns and architecture for implementing new API integrations in the Fync package. Follow these patterns precisely to ensure consistency and maintain the unified experience.

## 🏗️ Core Architecture Principles

### 1. **Functional-First Design**
- Use `function` declarations exclusively - NO arrow functions
- All functions must be pure where possible
- Prefer immutable data patterns
- Use composition over inheritance

### 2. **TypeScript Standards**
- Use `type` declarations exclusively - NO interfaces
- All type names MUST be prefixed with `T` (e.g., `TUser`, `TApiConfig`)
- Single non-exported types in component files should be named `TProps`
- Use named exports only (except for pages/views in Next.js projects)

### 3. **Code Style**
- Self-explanatory, readable code without comments
- NO inline or block comments unless absolutely necessary for obscure syntax
- Use descriptive function and variable names instead of comments
- If you feel the need to comment, rewrite for clarity instead

## 📁 File Structure Pattern

Every API integration follows this EXACT structure:

```
src/
├── [api-name]/
│   ├── index.ts          # Main API implementation
│   ├── types.ts          # API-specific types (optional)
│   └── README.md         # Usage documentation (optional)
└── core/                 # Shared architecture (DO NOT MODIFY)
```

## 🔧 Implementation Pattern

### Step 1: Define API Constants

```typescript
const [API_NAME]_API_BASE = "https://api.example.com/v1";
```

### Step 2: Define Resources Using `defineResource`

Each logical grouping of endpoints becomes a resource:

```typescript
const userResource = defineResource({
    name: "users",
    basePath: "/users",
    methods: {
        getUser: { path: "/{id}" },
        updateUser: { path: "/{id}", method: "PUT" },
        deleteUser: { path: "/{id}", method: "DELETE" },
        createUser: { path: "", method: "POST" },
    },
});
```

#### Resource Rules:
- **name**: Plural noun (users, repos, playlists)
- **basePath**: API path prefix (starts with `/`)
- **methods**: Object with method name as key

#### Method Definition Rules:
- **path**: Endpoint path with `{param}` placeholders
- **method**: HTTP method (defaults to "GET")
- **transform**: Optional response transformation function

### Step 3: Create Resources Object

```typescript
const resources = {
    users: userResource,
    posts: postResource,
    search: searchResource,
};
```

### Step 4: Create API Builder

```typescript
const build[ApiName] = createApiBuilder({
    baseUrl: [API_NAME]_API_BASE,
    auth: { type: "bearer" as const },
    headers: {
        "Content-Type": "application/json",
        // API-specific headers
    },
});
```

#### Auth Types:
- `bearer` - Authorization: Bearer {token}
- `basic` - Authorization: Basic {encoded}
- `apikey` - X-API-Key: {key}
- `oauth2` - Authorization: Bearer {token}

### Step 5: Define Module Type

```typescript
type T[ApiName]Module = TModule<typeof resources> & {
    // Helper method signatures
    getUser: (id: string) => Promise<any>;
    searchItems: (query: string, options?: any) => Promise<any>;
    // ... other convenience methods
};
```

### Step 6: Implement Main Export Function

```typescript
export function [ApiName](config: { token: string }): T[ApiName]Module {
    const base = build[ApiName](config, resources);
    const [apiName] = base as T[ApiName]Module;

    // Implement convenience methods
    [apiName].getUser = function (id: string) {
        return base.users.getUser({ id });
    };

    [apiName].searchItems = function (query: string, options?: any) {
        return base.search.search({
            q: query,
            limit: options?.limit || 20,
            ...options,
        });
    };

    return [apiName];
}
```

## 📋 Complete Implementation Template

Use this as your starting template for any new API:

```typescript
import { createApiBuilder, defineResource, type TModule } from "../core";

const [API_NAME]_API_BASE = "https://api.example.com/v1";

// Define all resources
const exampleResource = defineResource({
    name: "examples",
    basePath: "/examples",
    methods: {
        getExample: { path: "/{id}" },
        getExamples: { path: "" },
        createExample: { path: "", method: "POST" },
        updateExample: { path: "/{id}", method: "PUT" },
        deleteExample: { path: "/{id}", method: "DELETE" },
    },
});

const resources = {
    examples: exampleResource,
};

const build[ApiName] = createApiBuilder({
    baseUrl: [API_NAME]_API_BASE,
    auth: { type: "bearer" as const },
    headers: {
        "Content-Type": "application/json",
    },
});

type T[ApiName]Module = TModule<typeof resources> & {
    // Convenience method signatures
    getExample: (id: string) => Promise<any>;
};

export function [ApiName](config: { token: string }): T[ApiName]Module {
    const base = build[ApiName](config, resources);
    const [apiName] = base as T[ApiName]Module;

    // Implement convenience methods
    [apiName].getExample = function (id: string) {
        return base.examples.getExample({ id });
    };

    return [apiName];
}
```

## 🔧 Parameter Handling Patterns

### Path Parameters
Use `{param}` syntax in paths, pass in options object:

```typescript
// Resource definition
{ path: "/users/{userId}/posts/{postId}" }

// Usage
base.posts.getPost({ userId: "123", postId: "456" })
// Results in: GET /users/123/posts/456
```

### Query Parameters
Pass parameters that don't match `{param}` placeholders as query params:

```typescript
// Usage
base.search.search({ 
    q: "query",      // becomes ?q=query
    limit: 20,       // becomes &limit=20
    type: "user"     // becomes &type=user
})
```

### Request Bodies (POST/PUT/PATCH)
Pass data as first parameter, options as second:

```typescript
// For POST/PUT/PATCH methods
base.users.createUser(
    { name: "John", email: "john@example.com" }, // request body
    { team_id: "123" }                          // path/query params
);
```

## 📦 Package.json Updates

When adding a new API, update the exports in `package.json`:

```json
"./[api-name]": {
    "import": "./dist/src/[api-name]/index.js",
    "require": "./dist-cjs/src/[api-name]/index.js", 
    "types": "./dist/src/[api-name]/index.d.ts"
}
```

Also update keywords array with relevant terms for the new API.

## 📄 Main Index Export

Add your new API to `src/index.ts`:

```typescript
export { [ApiName] } from "./[api-name]";
export * from "./[api-name]/types"; // if types.ts exists
```

## ✅ Quality Checklist

Before submitting any new API integration, verify:

- [ ] Follows exact file structure pattern
- [ ] Uses `defineResource` for all endpoint groups
- [ ] All types prefixed with `T`
- [ ] Uses function declarations (no arrow functions)
- [ ] No comments unless absolutely necessary
- [ ] Convenience methods implemented for common use cases
- [ ] Package.json exports updated
- [ ] Main index.ts updated
- [ ] Auth configuration matches API requirements
- [ ] Base URL constant defined
- [ ] Resources object created with descriptive names
- [ ] Module type extends `TModule<typeof resources>`
- [ ] Export function matches naming pattern

## 🎯 Common Patterns & Examples

### Authentication Headers
```typescript
// Bearer token (most common)
auth: { type: "bearer" as const }

// API Key in header
auth: { type: "apikey" as const }

// Basic auth
auth: { type: "basic" as const }

// Custom headers (if needed)
headers: {
    "X-Custom-Header": "value",
    "Accept": "application/vnd.api+json",
}
```

### Path Interpolation Examples
```typescript
// Single param
{ path: "/users/{id}" }
// Usage: { id: "123" } → /users/123

// Multiple params  
{ path: "/orgs/{org}/repos/{repo}" }
// Usage: { org: "github", repo: "docs" } → /orgs/github/repos/docs

// No params
{ path: "/search" }
// Usage: { q: "query" } → /search?q=query
```

### Response Transformation
```typescript
{
    path: "/users/{id}",
    transform: (response: any) => ({
        id: response.user_id,
        name: response.full_name,
        email: response.email_address,
    })
}
```

## 🚫 What NOT to Do

- ❌ Don't use arrow functions anywhere
- ❌ Don't use `interface` - use `type` only
- ❌ Don't add comments unless absolutely necessary
- ❌ Don't modify core files unless adding general functionality
- ❌ Don't use default exports (except pages/views)
- ❌ Don't implement complex business logic in API layer
- ❌ Don't add heavy dependencies
- ❌ Don't deviate from the established patterns

## 🎯 API Recommendations

For APIs that fit perfectly with our patterns:
- **REST APIs** with standard CRUD operations
- **Token-based authentication**
- **JSON request/response**
- **Clear resource hierarchies**

Avoid APIs with:
- Complex authentication flows (OAuth callbacks, etc.)
- Real-time features (WebSockets, SSE)
- File upload/download as primary features
- Heavy SDK requirements

This architecture ensures all APIs feel consistent, performant, and maintainable while keeping the package lightweight and developer-friendly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/remcostoeten) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
