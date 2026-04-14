## documindintelliocr

> **CRITICAL**: Never rely on training data for library-specific code.

# Context7 Integration Rules

## Core Principle: Always Use Latest Documentation
**CRITICAL**: Never rely on training data for library-specific code.
 Always use Context7 MCP server to fetch current, version-specific documentation before generating any code that depends on external libraries.

## When to Use Context7
You MUST use Context7 (`use context7`) in the following scenarios:

### Required Libraries (Always Use Context7)
- **Next.js**: App Router, API routes, middleware, configuration, deployment
- **Supabase**: Auth, database operations, real-time subscriptions, storage, edge functions
- **React**: Hooks, components, state management, context API, concurrent features
- **TypeScript**: Advanced types, utility types, configuration, latest syntax
- **Tailwind CSS**: Classes, configuration, plugins, responsive design patterns
- **Prisma**: Schema, migrations, client operations, advanced queries
- **tRPC**: Routers, procedures, client setup, error handling
- **Zod**: Schema validation, type inference, error handling
- **Stripe**: Payment processing, webhooks, subscription management
- **Auth0/Clerk**: Authentication flows, middleware, session management
- **Vercel**: Deployment, edge functions, analytics, environment configuration

### Framework-Specific Patterns
Before implementing any of these patterns, use Context7:
- Authentication flows and middleware
- Database schema design and migrations
- API route implementations
- State management patterns
- Form validation and submission
- File upload and storage
- Real-time features
- Deployment and environment setup
- Testing configurations
- Performance optimization

## Implementation Protocol

### 1. Before Any Library-Related Code
```
Before implementing [specific feature] with [library name], let me get the latest documentation.

use context7

[Your original request]
```

### 2. For Updates or Debugging
When encountering library-related errors or when updating existing code:
```
I'm getting an error with [library]. Let me check the latest documentation before proposing a fix.

use context7

[Describe the error and context]
```

### 3. For Version-Specific Features
```
I need to implement [feature] using [library]. Let me ensure I'm using the most current approach.

use context7

[Feature requirements]
```

## Error Prevention Rules

### Deprecated Patterns to Avoid
- DO NOT suggest patterns without first checking Context7
- DO NOT assume API signatures from training data
- DO NOT use outdated configuration formats
- DO NOT implement features that may have been superseded

### Quality Assurance
- Always verify import statements against latest docs
- Confirm configuration syntax matches current version
- Check for breaking changes in recent releases
- Validate that suggested patterns follow current best practices

## Specific Library Guidelines

### Next.js
- Always use Context7 for: App Router patterns, Server Actions, middleware, API routes, deployment configuration
- Verify current patterns for: Data fetching, caching, routing, optimization

### Supabase
- Always use Context7 for: Auth helpers, database queries, real-time subscriptions, storage operations
- Check latest patterns for: Row Level Security, Edge Functions, local development setup

### React + TypeScript
- Always use Context7 for: Latest hook patterns, TypeScript integration, performance optimizations
- Verify current approaches for: State management, context usage, component patterns

## Workflow Integration

### Standard Development Flow
1. **Analyze Request**: Identify all libraries involved
2. **Context7 Check**: Use Context7 for each library mentioned
3. **Implementation**: Generate code using latest documentation
4. **Validation**: Cross-reference with official docs if needed

### Code Review Protocol
When reviewing or updating existing code:
1. Identify outdated patterns
2. Use Context7 to get current best practices
3. Suggest modern alternatives
4. Explain migration path if needed

## Exception Handling
Only skip Context7 when:
- Working with standard JavaScript/TypeScript language features
- Using well-established, stable APIs (DOM, Node.js core modules)
- Working with custom/internal code not dependent on external libraries

## Response Format
When using Context7, structure responses as:
1. **Documentation Check**: "Let me get the latest [library] documentation..."
2. **Current Approach**: Explain the modern, up-to-date pattern
3. **Implementation**: Provide code using latest syntax/patterns
4. **Notes**: Highlight any recent changes or important considerations

## Team Collaboration
- All team members must follow this Context7 protocol
- When pair programming, remind partner to use Context7 for library code
- Document any library-specific patterns discovered through Context7
- Share Context7 findings that reveal significant changes from common practices

Remember: It's better to spend 30 seconds getting current documentation than 30 minutes debugging outdated code patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-chun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
