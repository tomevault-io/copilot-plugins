## coolify-mcp

> FOLLOW Coolify MCP workflow WHEN implementing new Coolify API endpoints TO ensure consistent and tested MCP


# Coolify MCP Development Workflow

## Context
- When implementing new Coolify API endpoints as MCP tools
- When adding new features from the feature documentation
- When extending the MCP server capabilities

## Requirements
1. Start by reviewing feature documentation in docs/features
2. Verify endpoint exists in docs/openapi-chunks/* 
3. Follow existing implementation patterns:
   - Add types to src/types/coolify.ts
   - Add client method to src/lib/coolify-client.ts
   - Add MCP tool to src/lib/mcp-server.ts
   - Add simple test in src/__tests__/mcp-server.test.ts
4. Maintain strict TypeScript standards:
   - Always include explicit return types on functions
   - Import and use proper types from coolify.ts
   - No implicit any types
   - Follow existing type patterns
5. Keep tests focused and properly mocked:
   ```typescript
   it('should call client methodName', async () => {
     // Mock the method before calling it
     const mockResponse = { /* expected shape */ };
     const spy = jest.spyOn(server['client'], 'methodName')
       .mockResolvedValue(mockResponse);

     await server.method_name('test-uuid');
     expect(spy).toHaveBeenCalledWith('test-uuid');
   });
   ```
6. Follow gitflow as per 801-feature-workflow

## Examples
<example>
# Good implementation
1. Review docs/features/003-server-domains.md
2. Verify /servers/{uuid}/domains in openapi.yaml
3. Add ServerDomain interface to types
4. Add getServerDomains() to client with proper types:
   ```typescript
   async getServerDomains(uuid: string): Promise<ServerDomain[]> {
     return this.request<ServerDomain[]>(`/servers/${uuid}/domains`);
   }
   ```
5. Add get_server_domains tool to MCP server
6. Add properly mocked test:
   ```typescript
   const mockDomains = [{ ip: '1.2.3.4', domains: ['test.com'] }];
   const spy = jest.spyOn(client, 'getServerDomains')
     .mockResolvedValue(mockDomains);
   ```
7. Follow gitflow for commits and PR
</example>

<example type="invalid">
# Poor implementation
1. Start coding without checking docs
2. Add endpoint not in OpenAPI spec
3. Create new patterns different from existing code
4. Skip type definitions or use implicit any
5. Write tests that make real HTTP calls
6. Skip gitflow process
</example>

<critical>
- ALWAYS verify endpoint exists in OpenAPI spec
- ALWAYS include explicit return types on functions
- ALWAYS mock HTTP calls in tests
- NEVER introduce new patterns - follow existing code
- Keep tests simple and fully mocked
- Follow established file organization
- Fix ALL linting errors before committing
</critical> 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StuMason)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/StuMason)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
