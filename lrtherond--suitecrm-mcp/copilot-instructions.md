## suitecrm-mcp

> This document provides context and guidelines for AI assistants working on this SuiteCRM Model Context Protocol (MCP) server project.

# Claude Code Guidelines for SuiteCRM MCP Server

This document provides context and guidelines for AI assistants working on this SuiteCRM Model Context Protocol (MCP) server project.

## Project Overview

This is a TypeScript-based MCP server that integrates with SuiteCRM's JSON API (v8) to provide AI assistants with CRM capabilities. The server exposes tools for managing leads, contacts, and other CRM operations through the Model Context Protocol.

## Architecture

### Core Components

- **SuiteCRMClient**: Main API client handling authentication and HTTP requests
- **SuiteCRMLeadsClient**: Specialized client for lead management operations
- **SuiteCRMContactsClient**: Specialized client for contact management operations
- **SuiteCRMModule**: Generic module wrapper for CRUD operations
- **MCP Server**: Protocol server exposing tools to AI assistants

### Key Files

- `index.ts`: Main entry point with all classes and MCP server setup (2,184 lines)
- `test.ts`: Test suite for validating functionality
- `package.json`: Dependencies and build configuration
- `tsconfig.json`: TypeScript configuration
- `Dockerfile`: Container configuration

## Development Guidelines

### Code Organization

The project currently has all code in `index.ts`. For future refactoring:

1. **Client Classes** should be in separate files:
   - `src/clients/SuiteCRMClient.ts`
   - `src/clients/SuiteCRMLeadsClient.ts` 
   - `src/clients/SuiteCRMContactsClient.ts`

2. **Schemas** should be in `src/schemas/`:
   - `src/schemas/leads.ts`
   - `src/schemas/contacts.ts`
   - `src/schemas/base.ts`

3. **Tools** should be in `src/tools/`:
   - `src/tools/leadTools.ts`
   - `src/tools/contactTools.ts`
   - `src/tools/baseTools.ts`

### Environment Variables

Required environment variables (all must be set):
- `SUITECRM_URL`: SuiteCRM instance URL (include `/legacy`)
- `SUITECRM_USERNAME`: API user credentials
- `SUITECRM_PASSWORD`: API user password
- `SUITECRM_CLIENT_ID`: OAuth2 client ID
- `SUITECRM_CLIENT_SECRET`: OAuth2 client secret

### Testing

- Use `npm test` to run the test suite
- Tests create and clean up test data automatically
- Test credentials should come from environment variables only

### Build Process

```bash
npm run build      # Compile TypeScript
npm run watch      # Watch mode for development
npm run prepare    # Pre-publish build
```

### Docker Usage

The project includes a multi-stage Dockerfile for production deployment:
- Builder stage: Installs dependencies and compiles TypeScript
- Release stage: Production image with only runtime dependencies

## Common Tasks

### Adding New Tools

1. Define Zod schema for tool parameters
2. Add tool definition to appropriate tools array
3. Implement handler in the `CallToolRequestSchema` switch statement
4. Add corresponding method to appropriate client class
5. Update documentation in tool description

### Debugging

- Check `dist/server.log` for detailed API request/response logs
- Use extensive logging already implemented in `logToFile()` and `logRequestError()`
- All SuiteCRM API calls are logged with full request/response details

### Authentication Flow

The client handles OAuth2 flow automatically:
1. Initial login with username/password grant
2. Token storage and automatic refresh
3. Re-authentication when refresh tokens expire
4. All API calls include proper Authorization headers

## SuiteCRM Integration

### API Endpoints Used

- `Api/access_token`: OAuth2 authentication
- `Api/V8/module/{module}`: CRUD operations
- `Api/V8/module/{module}/{id}`: Single record operations
- `Api/V8/module/{module}/{id}/relationships/{relationship}`: Relationship operations
- `Api/V8/meta/modules`: Module metadata
- `Api/V8/meta/fields/{module}`: Field definitions

### Data Sanitization

The `sanitizeAttributes()` function removes read-only fields before create/update:
- `id`, `date_entered`, `date_modified`
- `modified_user_id`, `created_by`, `created_by_name`
- `lead_id` (for leads)

### Error Handling

- Zod validation for all tool inputs
- Comprehensive error logging with stack traces
- Graceful handling of SuiteCRM API errors
- Proper error responses to MCP clients

## Security Considerations

### Credentials

- Never commit credentials to version control
- Use environment variables for all sensitive data
- Validate all environment variables are present at startup
- Log requests/responses but sanitize sensitive data

### Input Validation

- All tool inputs validated with Zod schemas
- Proper type checking throughout
- Sanitization of user inputs before API calls

### API Security

- OAuth2 authentication with proper token management
- HTTPS recommended for SuiteCRM endpoints
- Minimal permissions for API user accounts

## MCP Protocol Implementation

### Tool Categories

1. **Generic Tools**: Work with any SuiteCRM module
2. **Lead-Specific Tools**: Enhanced lead management features
3. **Contact-Specific Tools**: Advanced contact operations

### Response Format

All tools return JSON responses with:
```json
{
  "content": [{"type": "text", "text": "JSON response"}]
}
```

Error responses include:
```json
{
  "content": [{"type": "text", "text": "Error message"}],
  "isError": true
}
```

## Performance Considerations

### Pagination

- Default page size: 20 records
- Maximum recommended: 100 records per request
- Use proper pagination for large datasets

### Caching

- OAuth tokens cached until expiry
- No data caching implemented (real-time data preferred)
- Consider implementing field metadata caching

### Rate Limiting

- No built-in rate limiting
- SuiteCRM instance may have limits
- Consider implementing backoff strategies for production

## Known Limitations

1. **Single File Architecture**: All code in one 2,184-line file
2. **No Module Discovery**: Hardcoded module support (Leads, Contacts, Accounts, Opportunities)
3. **Limited Relationship Support**: Basic relationship operations only
4. **No Bulk Operations**: Single record operations only
5. **No Advanced Filtering**: Basic JSON API filtering

## Future Enhancements

### High Priority
- Modularize codebase into separate files
- Add comprehensive input validation
- Implement consistent error handling patterns
- Add more SuiteCRM modules support

### Medium Priority  
- Add bulk operations support
- Implement advanced filtering and search
- Add data caching for metadata
- Improve relationship management

### Low Priority
- Add GraphQL support
- Implement webhook notifications
- Add audit logging
- Performance monitoring

## Troubleshooting

### Common Issues

1. **Authentication Failures**: Check credentials and OAuth client setup
2. **Module Not Found**: Verify module exists and user has permissions
3. **Field Validation Errors**: Check field names and types against SuiteCRM
4. **Network Errors**: Verify SuiteCRM URL and network connectivity

### Debug Steps

1. Check environment variables are set correctly
2. Review `dist/server.log` for API request/response details
3. Test SuiteCRM API directly with curl/Postman
4. Verify user permissions in SuiteCRM admin
5. Check OAuth client configuration

## Testing Guidelines

### Unit Testing
- Test each client method independently
- Mock SuiteCRM API responses
- Validate input/output schemas

### Integration Testing
- Test against real SuiteCRM instance
- Use dedicated test environment
- Clean up test data after runs

### MCP Testing
- Use MCP Inspector for tool testing
- Validate tool schemas and responses
- Test error handling scenarios

---

**Note**: This project is actively maintained. When making changes, ensure all tests pass and documentation is updated accordingly.

---
> Source: [lrtherond/SuiteCRM_MCP](https://github.com/lrtherond/SuiteCRM_MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
