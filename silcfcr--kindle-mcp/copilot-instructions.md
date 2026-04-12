## kindle-mcp

> An MCP (Model Context Protocol) server that processes Kindle highlight exports and generates personalized book memory summaries, then pushes them to Notion. Built in Node.js + TypeScript.

# Kindle MCP — Project Context

## What This Is
An MCP (Model Context Protocol) server that processes Kindle highlight exports and generates personalized book memory summaries, then pushes them to Notion. Built in Node.js + TypeScript.

**Key constraint:** Zero external AI API calls. All reasoning happens inside the host model (Claude Desktop). The server only parses files, builds prompt packages, and writes to Notion.

---

## Current Status
- [x] Full MCP server built and working
- [x] Claude Desktop connected and tested end-to-end
- [x] Auto-setup service (no NOTION_DATABASE_ID needed by users)
- [x] First-run auto-setup via Notion MCP (no user interaction required)
- [x] Published to npm as `kindle-mcp` — current version **1.0.3**
- [ ] Submit to modelcontextprotocol/servers registry
- [ ] List on smithery.ai

---

## npm Package
- **Name:** `kindle-mcp`
- **Latest version:** `1.0.3`
- **URL:** https://www.npmjs.com/package/kindle-mcp
- Auth token saved in `~/.npmrc` — can publish directly with `npm publish`
- Classic tokens are deprecated as of Nov 2025 — use **Granular Access Token** with **Bypass 2FA** checked

---

## Project Structure
```
Kindle MCP/
├── src/
│   ├── index.ts                            # MCP server entry, tool registry
│   ├── types/
│   │   └── index.ts                        # Shared interfaces
│   ├── services/
│   │   ├── configService.ts                # ~/.kindle-mcp/config.json R/W
│   │   ├── notionSetupService.ts           # Auto find-or-create Notion DB, NotionSetupRequiredError
│   │   └── notionService.ts               # Notion API page creation (blocks)
│   └── tools/
│       ├── processKindleExport.ts          # PRIMARY: parse + prompt build + auto DB
│       ├── initializeNotionDatabase.ts     # Creates DB under a given page ID, saves to config
│       ├── parseKindleClippings.ts         # HTML + plain text parser (auto-detects format)
│       ├── generatePersonalSummary.ts      # Prompt package builder (no API call)
│       └── pushToNotion.ts                # Notion write tool
├── dist/                                   # Compiled output (shebang + chmod 755)
├── .env                                    # Local secrets (never commit)
├── .env.example
├── package.json
├── tsconfig.json
├── CLAUDE.md                               # This file
└── README.md
```

---

## MCP Tools (5 total)

### 1. `process_kindle_export` — PRIMARY
- Input: `rawText` (full HTML or .txt file content)
- Parses highlights, resolves Notion DB ID, returns prompt packages + instructions
- If no DB found and no accessible pages → returns `status: "setup_required"` with instructions for Claude to auto-create via Notion MCP
- **Host model must call `push_to_notion` after generating — never display summary in chat**

### 2. `initialize_notion_database`
- Input: `parentPageId` (page ID returned by Notion MCP `notion-create-pages`)
- Creates "Kindle Book Summaries" DB under that page
- Saves DB ID to `~/.kindle-mcp/config.json`
- Called automatically by Claude during first-run setup

### 3. `push_to_notion`
- Input: `notionDatabaseId`, `summary` object (with optional `title` + `author`)
- Creates a Notion page with callout + heading/bullet blocks
- Returns `{ id, url, created_time }`

### 4. `parse_kindle_clippings`
- Input: `rawText`
- Auto-detects HTML export vs plain text My Clippings.txt
- Returns `{ books: [{ title, author, highlights[] }] }`

### 5. `generate_personal_summary`
- Input: `{ title, author, highlights[] }`
- Returns prompt package for host model (no API call)

---

## First-Run Auto-Setup Flow (New Users)
When a user has no existing DB and no accessible Notion pages:

```
process_kindle_export → status: "setup_required"
  ↓
Claude calls notion-create-pages (NO parent = workspace-level page)
  ↓
Claude calls initialize_notion_database(pageId)
  ↓
Claude calls process_kindle_export again (same rawText)
  ↓
Claude generates summaries → push_to_notion → returns Notion URL
```
Zero user interaction. Requires user to have Notion MCP running alongside kindle-mcp.

---

## Kindle File Formats Supported
1. **HTML export** — from Kindle app "Export Notebook" (`.html` file)
   - Detects via `<!DOCTYPE` / `<html` / `class="bookTitle"`
   - Skips bookmarks (`Marcador` / `Bookmark`)
   - Decodes HTML entities (no external deps)
2. **Plain text** — `My Clippings.txt` from Kindle device
   - Splits on `==========` separator

---

## Environment Variables
| Var | Required | Notes |
|---|---|---|
| `NOTION_API_KEY` | **Yes** | Only thing users need to provide |
| `NOTION_DATABASE_ID` | No | Auto-created if missing, saved to `~/.kindle-mcp/config.json` |

---

## Local Dev Setup
- **Claude Desktop config:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- Node path must be `/opt/homebrew/bin/node` — Claude Desktop does not inherit shell PATH
- To test with local build instead of npx:
```json
"kindle-mcp": {
  "command": "/opt/homebrew/bin/node",
  "args": ["/path/to/Kindle MCP/dist/index.js"],
  "env": { "NOTION_API_KEY": "secret_..." }
}
```
- To reset and test as a new user: delete `~/.kindle-mcp/` and remove `NOTION_DATABASE_ID` from env

---

## Public User Config (what users add to Claude Desktop)
```json
"kindle-mcp": {
  "command": "npx",
  "args": ["-y", "kindle-mcp"],
  "env": { "NOTION_API_KEY": "secret_..." }
}
```
Users also need the **Notion MCP** running for first-run auto-setup to work.

---

## How It Works End-to-End (Returning User)
1. User opens Claude Desktop
2. Attaches Kindle HTML export file
3. Says: "Use the process_kindle_export tool on this file"
4. Claude calls `process_kindle_export` → gets prompt packages + Notion DB ID
5. Claude generates summary JSON internally (no external API)
6. Claude calls `push_to_notion` → page created in Notion
7. Claude returns only the Notion page URL

---

## Build & Publish Commands
```bash
npm run build       # tsc + shebang + chmod 755
npm start           # run compiled server
npm run dev         # ts-node (no build)
npm publish         # publish to npm (token already saved in ~/.npmrc)
```
When publishing, bump version in `package.json` first.

---

## Remaining Distribution Steps
1. Submit PR to **github.com/modelcontextprotocol/servers** (official registry)
2. List on **smithery.ai** — add `smithery.yaml` to package first

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Silcfcr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Silcfcr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
