## strava-mcp

> This is a Model Context Protocol (MCP) server for Strava API integration built with TypeScript and Node.js.

# Strava MCP Server - Cursor Rules

## Project Overview
This is a Model Context Protocol (MCP) server for Strava API integration built with TypeScript and Node.js.

## Tech Stack
- **Runtime**: Node.js 18+ (LTS)
- **Language**: TypeScript with strict type checking
- **HTTP Client**: Native fetch API (no axios)
- **Validation**: Zod schemas
- **MCP SDK**: @modelcontextprotocol/sdk
- **Testing**: Jest
- **Linting**: ESLint with TypeScript plugin

## Code Style Guidelines

### TypeScript
- Use strict mode with all compiler options enabled (noImplicitAny, noUnusedLocals, noUnusedParameters, etc.)
- Always use explicit types for function parameters and return values
- Use `const` for immutable values, avoid `var`
- Prefer async/await over Promise chains
- Use ES modules (import/export) not CommonJS (require/module.exports)

### API Client
- All HTTP requests use native `fetch` (not axios)
- Always check `response.ok` before parsing JSON
- Use proper error handling with try/catch
- Authentication tokens are managed by the StravaAuth class

### Tool Definitions
- Each tool must have:
  - `description`: Clear description of what the tool does
  - `inputSchema`: Zod schema that will be converted to JSON Schema
  - `handler`: Async function that returns MCP-formatted response
- Tool responses must follow MCP format:
  ```typescript
  return {
    content: [
      {
        type: 'text' as const,
        text: JSON.stringify(data, null, 2),
      },
    ],
  };
  ```

### File Organization
```
src/
├── index.ts              # Main MCP server entry point
├── auth.ts               # OAuth 2.0 authentication logic
├── strava-client.ts      # Strava API client with all endpoints
├── types/
│   └── strava.ts         # TypeScript types for Strava API
└── tools/                # MCP tool definitions (one file per category)
    ├── activities.ts
    ├── athlete.ts
    ├── routes.ts
    ├── segments.ts
    ├── clubs.ts
    └── gear.ts
```

## Key Patterns

### Adding New Tools
1. Add the API method to `StravaClient` class in `strava-client.ts`
2. Create or update the appropriate tool file in `src/tools/`
3. Define Zod schema for input validation
4. Create handler that calls the client method
5. Export from the tool creator function
6. Tool will be automatically registered in `index.ts`

### Adding New API Endpoints
1. Add TypeScript types to `src/types/strava.ts` if needed
2. Add method to `StravaClient` class
3. Use the private `request<T>()` method for authenticated calls
4. Return properly typed responses

### OAuth Token Management
- Tokens are automatically refreshed by `StravaAuth.getValidAccessToken()`
- Never manually handle token refresh in API calls
- The auth class adds a 5-minute buffer before token expiration

## Testing
- Run tests with: `npm test`
- Use Jest with ts-jest for TypeScript support
- Mock fetch calls in tests
- Test both success and error paths

## Common Commands
- `npm run setup` - Interactive OAuth setup
- `npm run build` - Compile TypeScript
- `npm run dev` - Watch mode for development
- `npm run lint` - Run ESLint
- `npm test` - Run tests

## Important Notes
- Never commit `.env` file (contains secrets)
- Always use `zodToJsonSchema()` when registering tools
- The server runs on stdio transport for MCP
- All API calls go through the authenticated `request()` method
- Activity types use const array for type safety and enum validation

## Environment Variables
Required in `.env`:
- `STRAVA_CLIENT_ID` - From Strava API settings
- `STRAVA_CLIENT_SECRET` - From Strava API settings
- `STRAVA_ACCESS_TOKEN` - Obtained via OAuth flow
- `STRAVA_REFRESH_TOKEN` - Obtained via OAuth flow
- `STRAVA_EXPIRES_AT` - Unix timestamp of token expiration

## Strava API Reference
- Docs: https://developers.strava.com/docs/reference/
- Rate limits: 100 requests/15min, 1000 requests/day
- Base URL: https://www.strava.com/api/v3
- OAuth: https://www.strava.com/oauth/authorize

## Common Issues
- If tools don't show up: Check that `zodToJsonSchema()` is being called
- If 403 on OAuth: Check scope matches Strava app configuration
- If token errors: Run `npm run setup` to get fresh tokens

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gcoombe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
