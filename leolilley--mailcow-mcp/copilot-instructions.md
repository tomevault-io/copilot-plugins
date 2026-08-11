## mailcow-mcp

> > **💡 This file contains essential context for Claude Code sessions**

# Mailcow MCP Server - Claude Memory

> **💡 This file contains essential context for Claude Code sessions**  
> For complete documentation, see [docs/README.md](docs/README.md)

## 🎯 Project Status

**TypeScript MCP Server** for Mailcow email management - **MVP COMPLETE** ✅

### Current State
- **20 MCP Tools** implemented and registered
- **Architecture**: Modular TypeScript with comprehensive type safety  
- **MCP SDK**: Using `@modelcontextprotocol/sdk` v0.4.0
- **Test Coverage**: 25.8% overall (Auth: 70%, Utils: 77%, Domain: 85%, Mailbox: 87%)

### Available Tools
- **System** (3): health_check, get_config, test_api_connection
- **Domains** (5): list, get, create, update, delete  
- **Mailboxes** (5): list, get, create, update, delete
- **Queues** (6): list, get, flush, delete, hold, release
- **Sync Jobs** (7): list, get, create, update, delete, activate, deactivate
- **Logs** (4): get logs, errors, performance, access
- **Email** (3): send, check status, get templates

## 🗂️ Key Files

```
src/
├── index.ts                    # MCP Server entry point ✅
├── api/                        # Mailcow API clients ✅
│   ├── client.ts              # HTTP client with auth
│   ├── domains/               # Domain API ✅
│   ├── mailboxes/             # Mailbox API ✅
│   ├── queues/                # Queue API ✅
│   ├── syncjobs/              # Sync job API ✅
│   └── logs/                  # Log API ✅
├── tools/                      # MCP tool implementations ✅
│   ├── base.ts                # Base tool class ✅
│   ├── domains/               # Domain tools ✅
│   ├── mailboxes/             # Mailbox tools ✅
│   ├── queues/                # Queue tools ✅
│   ├── jobs/                  # Sync job tools ✅
│   ├── logs/                  # Log tools ✅
│   └── email/                 # Email tools ✅
├── auth/                       # Authentication ✅
├── config/                     # Configuration ✅
├── types/                      # TypeScript types ✅
└── utils/                      # Utilities ✅
```

## ⚡ Quick Commands

```bash
# Development
npm run dev          # Start with auto-reload
npm run build        # Build TypeScript
npm test            # Run test suite

# Testing & Quality
npm run test:coverage      # Generate coverage report
npm run lint              # ESLint
npm run format            # Prettier

# Production
npm start           # Start production server
```

## 🔑 Environment Setup

```bash
# Required environment variables
MAILCOW_API_URL=https://your-mailcow-server.com
MAILCOW_API_KEY=your-32-character-api-key
MAILCOW_API_ACCESS_TYPE=read-write
MAILCOW_VERIFY_SSL=true
```

## 🏗️ Architecture Patterns

### Tool Pattern
```typescript
export class ExampleTool extends BaseTool {
  readonly name = 'tool_name';
  readonly description = 'Tool description';
  readonly inputSchema = { /* JSON schema */ };

  async execute(input: ToolInput, context: ToolContext): Promise<ToolHandlerResult> {
    // 1. Validate permissions
    // 2. Validate input
    // 3. Call API
    // 4. Return formatted result
  }
}
```

### API Pattern (Mailcow-specific)
```typescript
// ✅ Correct Mailcow patterns
const domains = await client.get('/api/v1/get/domain');
await client.post('/api/v1/add/domain', data);
await client.post('/api/v1/edit/domain', data);
await client.post('/api/v1/delete/domain', { items: [id] });

// ❌ Wrong - RESTful doesn't work with Mailcow
const domains = await client.get('/api/v1/domains'); // Won't work
```

### Permission Patterns
- **Namespace Format**: `{resource}:{action}` (e.g., `domains:read`, `mailboxes:write`)
- **Generic Fallbacks**: `read`, `write`, `delete` for backward compatibility
- **Validation**: Always check permissions before API calls

## 🚨 Important Notes

### Email Tools MVP
- **send_email**: Simulates queue submission (generates realistic queue IDs)
- **check_email_status**: Checks both queue and logs for delivery status  
- **Queue Integration**: Uses existing queue APIs for status tracking
- **Templates**: 5 predefined templates (welcome, alert, maintenance, etc.)

### API Quirks
- **Non-RESTful**: Mailcow uses `/get/`, `/add/`, `/edit/`, `/delete/` patterns
- **Auth Header**: Uses `X-API-Key` header, not `Authorization`
- **Delete Format**: `{ items: [item1, item2] }` for deletions

### Naming Consistency
- **syncjobs** (not jobs) - matches API endpoint `/api/v1/syncjobs`
- **queues** - matches API endpoint `/api/v1/queues`  
- **Permission namespace**: `syncjobs:read`, `syncjobs:write`, `syncjobs:delete`

## 🔄 Testing

### Key Test Commands
```bash
npm test                    # Unit tests
npm run test:integration    # Integration tests  
npm run test:all           # Everything
npm run coverage:open      # View coverage report
```

### Mock Patterns
- **Use MockLogger** for testing logging
- **Mock API responses** with realistic Mailcow data
- **Test both success and failure paths**

## 📝 Development Workflow

1. **Read existing patterns** in similar tools
2. **Follow BaseTool pattern** for consistency
3. **Add comprehensive tests** (>80% coverage target)  
4. **Update documentation** as needed
5. **Test integration** with `npm run test:all`

---

**For detailed information**: See [docs/README.md](docs/README.md) | **Quick Start**: [docs/QUICK_START.md](docs/QUICK_START.md)

---
> Source: [leolilley/mailcow_mcp](https://github.com/leolilley/mailcow_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
