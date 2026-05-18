## adapters

> - Implement core port interfaces

# MCP Prompts Adapters Rules

## Implementation Guidelines
- Implement core port interfaces
- Handle all error cases gracefully
- Use proper async/await patterns
- Include integration tests

## Dependencies
- Can depend on @mcp-prompts/core
- External libraries as needed
- Follow adapter pattern strictly

## Testing Strategy
- Integration tests with real dependencies
- Mock external services when needed
- Test error handling extensively
- Performance tests for critical paths

## Build Configuration
- Own tsconfig.json extending root options
- Build outputs to dist/
- Include type declarations 

---
> Source: [sparesparrow/mcp-prompts](https://github.com/sparesparrow/mcp-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
