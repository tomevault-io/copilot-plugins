## mcp-server-currency-converter

> - Focus on unit testing service layer functions


# Testing and Development Guidelines

## Testing Approach
- Focus on unit testing service layer functions
- Mock external API calls to Brasil API
- Test both success and error scenarios
- Validate input parameter handling

## Development Workflow
- **Configuration Only**: Set up and configure features without running tests or starting servers
- **Manual Testing**: Leave manual testing and server startup to the user
- **Build First**: Always run `bun run build` after making changes

## API Testing
- Use tools like curl, Postman, or REST clients for manual API testing
- Test endpoints via Swagger UI at `/api-docs`
- Verify health check endpoint: `GET /health`

## MCP Testing
- Test MCP server integration through Claude Desktop App
- Verify tools appear in Claude interface after configuration
- Test actual tool execution with sample queries

## Error Scenarios to Test
- Invalid date formats
- Unsupported currencies
- Network failures
- Empty API responses
- Weekend/holiday dates with no data

## Documentation
- Keep README.md updated with latest usage examples
- Maintain Swagger documentation accuracy
- Update build and deployment instructions as needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gustavo-Kuze)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Gustavo-Kuze)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
