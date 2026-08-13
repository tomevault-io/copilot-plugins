## risk-alert

> This is a **Google Apps Script** project that automates security vulnerability alert processing via Gmail. It scans incoming emails for vulnerability warnings, matches them against an internal asset list, and creates appropriate draft responses.

# Risk Alert System - Copilot Instructions

## Project Overview
This is a **Google Apps Script** project that automates security vulnerability alert processing via Gmail. It scans incoming emails for vulnerability warnings, matches them against an internal asset list, and creates appropriate draft responses.

## Architecture

### Core Components
| File | Purpose |
|------|---------|
| `env.js` | Configuration constants (`CONFIG` object) - email addresses, spreadsheet IDs, webhook URLs |
| `code.js` | Main business logic - email processing, asset matching, logging, and web app endpoints |
| `Dashboard.html` | Frontend dashboard served via `doGet()` as a Google Apps Script Web App |

### Data Flow
1. **Email Ingestion**: `processIncomingEmails()` queries Gmail for unread emails matching sender + subject keyword
2. **Asset Matching**: Compares extracted warning info against Google Sheet asset list (columns A, B, C)
3. **Draft Creation**: Creates Gmail drafts - either to Person A (asset matched) or reply to sender (no match)
4. **Logging**: Records all processing results to `SystemLogs` sheet with deduplication via Message ID

### Google Sheets Structure
- **Sheet1**: Asset list (3 columns: A, B, C) for vulnerability keyword matching
- **Settings**: Toggle switches (A2: scan read emails, B2: auto-draft, C2: chat notify)
- **SystemLogs**: Execution history with columns: Timestamp, Status, Warning Name, Matched Asset, Action, Email Date, Message ID

## Key Patterns

### Frontend-Backend Communication
Dashboard uses `google.script.run` pattern:
```javascript
// Frontend call
google.script.run.withSuccessHandler(callback).getSystemSettings();

// Backend must expose functions globally (not inside objects)
function getSystemSettings() { ... }
```

### Regex Extraction
Warning info extracted using specific Chinese patterns:
```javascript
const nameRegex = /警訊名稱[：:]\s*(.+)/i;  // "警訊名稱：" prefix
const descRegex = /漏洞說明[：:]\s*(.+)/i;  // "漏洞說明：" prefix
```

### Settings Toggle Pattern
Settings stored as "是"/"否" strings in spreadsheet cells (Chinese for Yes/No).

## Development Notes

### Deployment
- Deploy as Web App in Google Apps Script editor
- `doGet()` serves the Dashboard.html as the entry point
- Functions called by frontend must be top-level (global scope)

### Configuration
Update `CONFIG` object in `env.js` before deployment:
- `SPREADSHEET_ID`: Target Google Sheet for assets and logs
- `SENDER_B_EMAIL`: Email address to filter incoming warnings
- `PERSON_A_EMAIL`: Recipient for asset-matched alerts
- `GOOGLE_CHAT_WEBHOOK_URL`: Optional Google Chat notifications

### Deduplication
Uses Gmail Message ID stored in column G of SystemLogs to prevent reprocessing same email.

## Common Modifications

- **Add new asset columns**: Update `fetchComparisonData()` range (currently reads 3 columns)
- **Change email template**: Modify `createDraftForPersonA()` or `createDraftReplyToSenderB()`
- **Add new dashboard controls**: Add switch in HTML + handler in `updateSystemSetting()`

---
> Source: [takahope/risk-alert](https://github.com/takahope/risk-alert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
