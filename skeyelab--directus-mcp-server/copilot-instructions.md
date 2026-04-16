## directus-mcp-server

> When adding new tools or features to this MCP server, always organize them into appropriate toolsets:

# Directus MCP Server Cursor Rules

## Toolset Organization

When adding new tools or features to this MCP server, always organize them into appropriate toolsets:

1. **Schema and Content tools must belong to the `default` toolset** - This ensures backward compatibility and allows clients to access core tools when no specific toolset is requested.

2. **Flow tools do NOT belong to the `default` toolset** - Flow tools are only in the `flow` toolset and must be explicitly requested.

3. **Assign tools to their specific toolset**:
   - Schema-related tools → `schema` toolset (collections, fields, relations, schema snapshots)
   - Content-related tools → `content` toolset (items CRUD, queries, bulk operations)
   - Flow-related tools → `flow` toolset (workflow automation, triggers) - NOT in default

4. **Toolset assignment format**:
   ```typescript
   // For schema and content tools:
   {
     name: 'tool_name',
     description: 'Tool description',
     inputSchema: z.object({...}),
     toolsets: ['default', 'schema'] as const,  // Always include 'default' + specific toolset
     handler: async (client, args) => {...}
   }
   
   // For flow tools:
   {
     name: 'tool_name',
     description: 'Tool description',
     inputSchema: z.object({...}),
     toolsets: ['flow'] as const,  // Only 'flow', NOT 'default'
     handler: async (client, args) => {...}
   }
   ```

5. **When creating new tool files**:
   - Determine which toolset the tools belong to based on their functionality
   - Schema and content tools: Add to both `default` and their specific toolset
   - Flow tools: Add ONLY to `flow` toolset (not `default`)
   - Update the toolset filtering logic in `src/index.ts` if adding a new toolset category

6. **Toolset types are defined in** `src/types/index.ts` as:
   ```typescript
   export type Toolset = 'default' | 'schema' | 'content' | 'flow';
   ```

This organization allows clients to filter tools using the `MCP_TOOLSETS` environment variable, improving AI agent accuracy by reducing tool confusion. The default toolset contains only schema and content tools, while flow tools must be explicitly requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Skeyelab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
