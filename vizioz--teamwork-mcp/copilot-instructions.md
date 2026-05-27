## add-new-teamwork-mcp-endpoint

> Step that I would like you to follow when adding a new MCP endpoint to this solution.


# Steps to Add a New Teamwork MCP Endpoint

NOTE: When implementing any API calls using `api.post()` the path must not include the base path which is: `/projects/api/v3` 

1 Use Swagger MCP to generate typescript models used by this endpoint to allow you to strongly type code with the correct models:

`mcp__listEndpointModels --swaggerFilePath [path from .swagger-mcp file] --path [API endpoint path] --method [HTTP method]`

2. Use Swapper MCP to generate the MCP Endpoint definition.

`mcp__generateEndpointToolCode --swaggerFilePath [path from .swagger-mcp file] --path [API endpoint path] --method [HTTP method]`

3 Create service implementation file:

- Create a new file in src/services/[category]/[endpointName].ts
- Implement the service function following existing patterns
- Export the function as both named export and default export

4 Update services/index.ts:

- Import the new service function
- Add it to the named exports
- Add it to the default export object

5 Create tool implementation file:

- Create a new file in src/tools/[category]/[endpointName].ts
- Check the naming conventions used in the other tools.
- Define the tool definition with name, description, and input schema
- Implement the handler function that calls the service

6 Update tools/index.ts:

- Import the tool definition and handler
- Add them to the toolPairs array
- Add the handler to the exports list

7 Update Todo.md:

- Mark the endpoint as completed (✅) in the appropriate section
- Add an entry in the "MCP Implementation Issues" section documenting the completion

8 Test the new endpoint:

- Create a test case that uses the new endpoint
- Verify the response is as expected
- If the endpoint is a DELETE, always create an item to delete, do not delete any existing content without user consent.

This workflow ensures that new endpoints are properly integrated into the existing codebase structure and follow the same patterns as other endpoints.

---
> Source: [Vizioz/Teamwork-MCP](https://github.com/Vizioz/Teamwork-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
