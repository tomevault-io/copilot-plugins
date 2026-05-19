## monday-com-tasks

> **If you see "Not Authenticated" errors, check for direct API calls!**

# Claude Code Context for forwarding-monday

## CRITICAL: Use core-api for ALL External APIs

**If you see "Not Authenticated" errors, check for direct API calls!**

This service MUST use core-api for all external API calls. Direct `fetch()` to Monday/Slack/Claude APIs will fail because API keys are only stored in core-api.

### Troubleshooting "Not Authenticated"
1. Find the code making the API call
2. Check if it uses direct `fetch('https://api.monday.com/...')`
3. Replace with: `import { monday } from './services/coreApi.js'` and `monday.query(...)`

---

## Core-API Migration (IMPORTANT)

### Objective
All external API calls (Monday.com, Slack, Claude AI, Google APIs) should go through **core-api** instead of making direct API calls from this service.

### Why?
1. **Centralized authentication** - API keys stored in one place (core-api), not scattered across services
2. **Rate limiting** - core-api handles rate limits and retries for all services
3. **Easier maintenance** - Update API integrations in one place
4. **Security** - Services don't need individual API keys

### Architecture
```
┌─────────────────────┐     ┌─────────────────────┐
│  forwarding-monday  │────▶│      core-api       │────▶ Monday.com
│                     │     │                     │────▶ Slack
│  ta-pipeline-mgr    │────▶│  (centralized hub)  │────▶ Claude AI
│                     │     │                     │────▶ Google APIs
│  other services...  │────▶│                     │────▶ ConvertAPI
└─────────────────────┘     └─────────────────────┘
```

### How to Use core-api

**Import the client:**
```typescript
import { monday, slack, claude, google } from './services/coreApi.js';
```

**Make API calls:**
```typescript
// Monday query
const result = await monday.query(graphqlQuery, { boardId: '123' });

// Slack message
await slack.postMessage({ channel: 'C123', text: 'Hello' });

// Claude analysis
const analysis = await claude.analyze({ content: 'text', systemPrompt: '...' });
```

### DO NOT
- Make direct `fetch()` calls to `api.monday.com`
- Make direct `fetch()` calls to Slack API
- Make direct `fetch()` calls to `api.anthropic.com`
- Store API keys in this service's environment variables (except CORE_API_KEY)

### Required Environment Variables
```bash
CORE_API_URL=http://core-api.railway.internal:8080
CORE_API_KEY=<shared-api-key>
```

### Boards & Channels (from core-api config)

**Monday.com Boards:**
| Key | Purpose |
|-----|---------|
| `seasonTicketTasks` | Main task board for forwarding-monday |

**Slack Channels:**
| Key | Purpose |
|-----|---------|
| `seasonTicketAdmin` | Main channel for task notifications |
| `issueCall` | Issue call digests |
| `control` | Control/admin notifications |
| `supporterPrimary` | Primary supporter channel |
| `supporterSecondary` | Secondary supporter channels (comma-separated) |

Access via: `import { getCachedConfig } from './services/coreApi.js'`
```typescript
const config = getCachedConfig();
const boardId = config.monday.boards.seasonTicketTasks;
const channelId = config.slack.channels.seasonTicketAdmin;
```

### Files Using core-api
- `src/services/coreApi.ts` - The client (already complete)
- `src/services/digest.ts` - Uses `monday.query()` for task fetching
- `src/services/monday.ts` - Should use core-api (may need migration)
- `src/services/slack.ts` - Should use core-api (may need migration)

### Migration Status
- [x] digest.ts - Migrated to core-api for Monday queries
- [x] sheets.ts - Migrated to core-api for all Sheets operations (create, batchUpdate, getMetadata, getValues, updateValues)
- [ ] monday.ts - Check if using direct API calls
- [ ] slack.ts - Check if using direct API calls
- [ ] Other files - Audit for direct API usage

## Project Overview
Email-to-Monday.com task management system with Slack integration and scheduled digests.

---
> Source: [edwardmsalem/monday.com-tasks](https://github.com/edwardmsalem/monday.com-tasks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
