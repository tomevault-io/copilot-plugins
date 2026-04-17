## things-mcp

> This is a Model Context Protocol (MCP) server that provides integration with the Things app via its URL scheme. It's built with TypeScript and uses the MCP SDK.

# Things MCP Server - Cursor Rules

## Project Overview

This is a Model Context Protocol (MCP) server that provides integration with the Things app via its URL scheme. It's built with TypeScript and uses the MCP SDK.

## Code Style & Standards

### TypeScript

- Use strict TypeScript configuration
- Prefer explicit types over `any`
- Use Zod for runtime type validation and schema definition
- Follow async/await patterns over Promise chains
- Use proper error handling with try/catch blocks

### MCP Server Patterns

- All tools should use Zod schemas for parameter validation
- Use descriptive parameter descriptions for better AI understanding
- Follow the MCP tool response format with content arrays
- Handle URL encoding properly using URLSearchParams
- Validate required parameters and provide clear error messages

### Things URL Scheme

- Always use the `things:///` protocol
- Properly encode all parameters using URLSearchParams
- Support optional parameters by checking for undefined/null
- Follow the official Things URL scheme documentation
- Handle both single and multiple item operations
- Support all parameter types: strings, arrays, booleans, dates

### Error Handling

- Wrap exec calls in try/catch blocks
- Provide meaningful error messages
- Handle edge cases like missing required parameters
- Validate data before constructing URLs

### Code Organization

- Keep tool definitions together and well-organized
- Use consistent parameter naming (kebab-case for URL params)
- Group related functionality logically
- Add comments for complex URL construction logic

### Testing Considerations

- Consider adding validation for Things app availability
- Test URL encoding with special characters
- Validate parameter combinations work correctly
- Test both success and error scenarios

## Development Guidelines

- Follow semantic versioning for releases
- Update README when adding new features
- Ensure all Things URL scheme features are supported
- Test with actual Things app when possible
- Keep dependencies minimal and up to date

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimfilippou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
