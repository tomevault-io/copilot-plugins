## mcp-starter-template

> This document provides instructions for AI agents working on this codebase.

# AI Agent Instructions for MCP Starter Template

This document provides instructions for AI agents working on this codebase.

## Project Vision
A minimal, highly structured, and modular MCP server template in TypeScript.

**Dual Role**:
1. **Editable Boilerplate**: Fork or clone this to scaffold new MCP servers rapidly.
2. **Reference Library**: Keep this template alongside your other MCP servers. It serves as a 2026 "Gold Standard" that agents can read anytime they need to build, edit, or improve an MCP server's tool orchestration and metadata.

## Mandatory Reading

> Before creating or modifying **any** tool, you **MUST** read and internalize [Tool Description Prompt Engineering](./docs/descriptions_prompt_engineering.md). It contains the canonical examples, patterns, and rules for 2026-grade tool metadata.

## Architecture Guidelines (Agent-First & Tools-First)
In 2026, **Autonomous AI Agents** are the primary consumers of MCP. Follow these principles:

- **Tools-First Strategy**: Focus on creating simple, efficient, and robust **Tools**. They are the most reliable primitive for agentic workflows.
- **Discouraged Primitives**: In 99% of cases, **Resources**, **Prompts**, and **Sampling** should be considered legacy or specialized edge cases.
- **Long-Running Processes**: Use tools with **Progress Logging** (`server.server.sendLoggingMessage`) for tasks that take time.
- **Transport & Registration**: Exclusively `stdio`. Use `registerTool` config-based registration for strict typing and runtime flexibility.
- **Sampling Fallbacks**: If sampling is absolutely necessary, **mandate** a manual tool fallback. Never assume client support.

## SDK v1.27.1 Native Features — Use Them, Don't Reinvent Them
The SDK provides first-class support for metadata and flow control that agents use to reason about tools. **Always** use these over custom approximations:

| Feature | Where | Purpose |
|---|---|---|
| `title` | `registerTool`, `registerPrompt`, `registerResource` | Human-readable display name. Distinct from the programmatic `name`. |
| `annotations` | `registerTool` config | Behavioral hints: `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`. Agents use these to assess risk and cacheability. |
| `outputSchema` | `registerTool` config | Zod schema for structured output validation. Use when returning typed data. |
| `ctx.signal` | Tool Handler Context | `AbortSignal` for standard cancellation. Inherit into fetchers or loops (`signal: ctx.signal`). |
| `server.server.sendLoggingMessage` | Tool Logic | Emit structured logs back to the client during long operations. |
| `.enable()` / `.disable()` | `RegisteredTool`, `RegisteredResource`, `RegisteredPrompt` | Toggle visibility at runtime. Call `server.server.sendToolListChanged()` after. |
| `.update()` | All registered objects | Modify name, description, schema, or callback after registration. |
| `.remove()` | All registered objects | Permanently unregister the primitive. |

**Every tool MUST have**: `title`, `description`, `annotations`, and fully `.describe()`'d `inputSchema`.

## Dynamic Tool Registration
**Dynamic Registration** is the single most efficient way to leverage MCP servers in 2026:

- **Context Rot Prevention**: Enable/disable tools based on session state to prevent irrelevant tools from polluting the agent's context.
- **Precision Orchestration**: Push the "right tool at the right time" (e.g., enable analysis tools only after data retrieval).
- **Implementation**: Use `RegisteredTool.enable()` / `.disable()` at runtime. Always call `server.sendToolListChanged()` to notify the client.

## Graduated Complexity: Tool Orchestration Levels
Always aim for the lowest complexity that achieves the objective. See `src/tools/examples.ts` for blueprints:

1. **Level 1: Atomic** — Single-purpose tools like `ping_host`. High reliability, no state.
2. **Level 2: Sequential** — Tools like `extract_keywords` designed to pipe into `search_knowledge_base`. Use descriptions to guide the dependency chain.
3. **Level 3: State-Aware** — Tools like `toggle_workspace_lock` that manage the agent's operating environment.

## Quick Start for New Servers

1. **Prune Primitives**: If Resources, Prompts, or Sampling aren't needed, **delete them immediately**.
   - Remove: `src/resources/`, `src/prompts/`.
   - Cleanup `src/server.ts` imports and registration calls.
2. **Focus on Tools**: Add your logic to `src/tools/`.
3. **Minimalist Architecture**: Prefer a small set of high-impact, versatile tools over a large library of niche ones.
4. **Agentic Trust**: Trust that modern agents are smart enough to choose the right tool from high-quality metadata. Design "Golden Path" tool sequences (Discovery → Analysis → Action).

## Crafting Tool Descriptions

Autonomous agents rely entirely on descriptions to know **when** and **how** to use a tool. The main description and Zod argument descriptions form a **unified interface** — they must be complementary, never redundant.

**Main Description** (the `description` field):
- Start with a clear intent verb (e.g., "Analyze", "Retrieve", "Toggle").
- Include "When to use" heuristics and "Returns" expectations.
- Add natural-language examples (e.g., "For example, if you need to check a service before calling it...").
- State constraints — what the tool *cannot* do.
- Hint at orchestration follow-ups (e.g., "After extracting keywords, use `search_knowledge_base`").

**Argument Descriptions** (Zod `.describe()`):
- **ALWAYS** include `.describe()` on every parameter — no exceptions.
- Include natural-language examples inline (e.g., `'The hostname to ping, for example google.com'`).

### Zod Schema Design: The Other Steering Mechanism
The Zod schema is not just validation — it is a **primary behavior-steering mechanism** alongside the tool description. An optimally constrained schema drastically reduces agent errors and hallucinated arguments.

**Schema Type Requirements (TS SDK v1.27.1)**:
The SDK has specific type expectations for different primitives. Mixing these will cause TypeScript compilation errors.

| Primitive | Field | Requirement | Reason |
|---|---|---|---|
| **Tools** | `inputSchema` | `z.object({...})` | Required for argument validation and handler type inference. |
| **Tools** | `outputSchema` | `{...}` (Raw Shape) | **Advanced**: Defines machine-readable `structuredContent`. Helps agents plan follow-up steps. |
| **Prompts** | `argsSchema` | `{...}` (Raw Shape) | SDK expects a raw shape to wrap the prompt arguments. |

**When to use `outputSchema`**:
- **90% of Tools**: Not needed. Standard `content` (text/image) is sufficient for most tasks.
- **10% High-Impact Tools**: Use for search, analysis, or state-toggles where the agent needs to "pipe" data into another tool or make a logic-based decision based on a specific field (e.g., a `confidence` score or `status` flag).
- **Agentic Planning**: It steers the agent's **post-call strategy** by explicitly defining the machine-readable contract of the result.

**Rules**:
- **MANDATORY ZOD V3**: Always use `zod@3.25+` (pin to the same version the SDK internally uses, usually `^3.25.0`). Do not use `zod@4`. The MCP SDK includes a backward-compatibility layer for v4 that breaks native TypeScript generic inference (`AnySchema` vs `ZodTypeAny` mismatches). Using `zod@3` directly ensures flawless type extraction.
- **Always `.describe()`**: Every single parameter must have a `.describe()` with clear intent and a natural-language example.
- **Enum over String**: If there are a known set of valid values, **always** use `z.enum()` instead of `z.string()`. This eliminates guesswork entirely.
- **Constrain Numerics**: Always use `.min()`, `.max()`, `.int()`, `.positive()` etc. to define valid ranges.
- **Defaults**: Use `.default()` generously. Agents should be able to call a tool with minimal arguments and get reasonable behavior.
- **Optional with Intent**: Use `.optional()` for truly optional parameters, but combine with `.describe()` to explain when to use it.
- **Output Types**: Prefer `.url()`, `.email()`, `.uuid()` etc. over raw `z.string()` when the format is known.

```typescript
// ❌ BAD: vague, unconstrained, no descriptions
inputSchema: {
    query: z.string(),
    count: z.number(),
    format: z.string(),
}

// ✅ GOOD: constrained, described, agent-friendly
inputSchema: z.object({
    query: z.string().describe('The search query, for example "react hooks best practices"'),
    count: z.number().int().min(1).max(50).default(10)
        .describe('Number of results to return.'),
    format: z.enum(['json', 'markdown', 'plain']).default('json')
        .describe('Output format. Use markdown when results will be shown to the user.'),
}),
outputSchema: {
    results: z.array(z.string()).describe('List of search result titles.')
}
```

**Anti-Instruction-Drift**: **NEVER** provide JSON samples or literal code snippets in the main description. Trust the Zod schema for the *how*; use descriptions for the *why* and *when*.

> For detailed patterns and official examples, see [Tool Description Prompt Engineering](./docs/descriptions_prompt_engineering.md).

## Testing

For every new capability, adapt `tests/integration/cli.test.ts`. Tests use the MCP Inspector CLI for black-box protocol validation.

```bash
# Build before testing
pnpm run build

# Run tests
pnpm test

# Manual inspection
npx @modelcontextprotocol/inspector --cli node dist/index.js

# List capabilities
npx @modelcontextprotocol/inspector --cli node dist/index.js --method tools/list

# Call a tool
npx @modelcontextprotocol/inspector --cli node dist/index.js --method tools/call --tool-name echo --tool-arg message="Hello World"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Git-Fg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
