## api-investigation

> title: API Endpoint Investigation from OpenAPI/Swagger

title: API Endpoint Investigation from OpenAPI/Swagger
when:
  - Facing persistent API errors (e.g., 404, 400) for a specific operation.
  - Suspecting a mismatch between implemented API calls and actual API specification.
  - An OpenAPI/Swagger specification file is available for the target API.
then:
  priority: High
  steps:
    - "Verify the API specification version being used (e.g., v2, v3) matches the intended API version."
    - "If the specification file is minified or single-line JSON, request or perform formatting (e.g., using `jq . original.json > formatted.json`) for easier manual inspection and searching."
    - "Use `grep_search` (or equivalent text search) within the formatted specification file to locate the operation in question. Search by:
        - `operationId` if known (e.g., `\"operationId\": \"createDoc\"`)
        - HTTP method and relevant path segments (e.g., `\"post\":`, `/docs`, `/workspaces/{workspaceId}/docs`)
        - Relevant tags (e.g., `\"tags\": [\"Docs\"]`)
        - Summary or description keywords."
    - "Once the operation definition is found, carefully review:
        - The exact `path` and ensure all path parameters are correctly supplied and typed (e.g., string vs. number).
        - The HTTP `method` (GET, POST, PUT, DELETE).
        - `parameters` (path, query, header): ensure all required parameters are included and correctly named. Pay attention to `in: \"path\"`, `in: \"query\"`, etc.
        - `requestBody`: ensure the structure matches the specification, especially `required` fields and data types. Note the `content-type` (e.g., `application/json`)."
    - "Update the client-side service implementation (e.g., `*.service.ts`) to precisely match the specification for the endpoint path, method, parameters, and request body."
    - "Update associated type definitions (e.g., `types.ts` for request/response DTOs) to reflect the specification."
    - "Update the tool definition's `inputSchema` (e.g., `*.tools.ts`) to correctly describe the parameters the user needs to provide, ensuring it aligns with the service layer's requirements."
    - "Iterate by rebuilding the MCP server and re-testing the specific tool call. Examine server logs for detailed error messages from the ClickUp API."
    - "If issues persist, consider that the live API might differ from the provided specification or that there are other undocumented requirements (e.g., feature flags, permissions, incorrect IDs for resources)."

---
> Source: [Nazruden/clickup-mcp-server](https://github.com/Nazruden/clickup-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
