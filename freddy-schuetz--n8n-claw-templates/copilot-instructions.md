## n8n-claw-templates

> This file helps AI assistants (Claude Code, Copilot, etc.) and human contributors create and contribute templates.

# CLAUDE.md — n8n-claw Templates

This file helps AI assistants (Claude Code, Copilot, etc.) and human contributors create and contribute templates.

## What is this repo?

A template catalog for [n8n-claw](https://github.com/freddy-schuetz/n8n-claw). Each template is a pre-built MCP server that can be installed via chat command. For the main project architecture, see the [n8n-claw CLAUDE.md](https://github.com/freddy-schuetz/n8n-claw/blob/main/CLAUDE.md).

## Template Structure

Templates come in two shapes — `native` and `bridge`.

**Native** (`type: "native"`) — n8n implements the tool logic itself. 3 files:

```
templates/
  index.json                    <- catalog (one entry per template)
  {template-id}/
    manifest.json               <- metadata (name, tools, credentials)
    workflow.json               <- n8n workflow bundle (sub + server)
```

**Bridge** (`type: "bridge"`) — registers an existing external MCP server (Streamable HTTP) into `mcp_registry`. **No `workflow.json`** — just `manifest.json` + `index.json` entry. Supported since n8n-claw v1.3.0. See [templates/TEMPLATE_EXAMPLE.md#bridge-templates](templates/TEMPLATE_EXAMPLE.md#bridge-templates) for the manifest schema (`bridge.mcp_url`, `auth_type`, `auth_token_required`, …) and [templates/deepwiki/](templates/deepwiki/) as a working no-auth example.

The rest of this document covers the **native** path. If you are shipping a bridge template, stop here and follow the `TEMPLATE_EXAMPLE.md` section above.

## Two-Workflow Pattern

Every **native** template contains **two workflows** in `workflow.json`:

1. **`sub`** — Sub-workflow with actual tool logic (`executeWorkflowTrigger` -> `Code` node)
2. **`server`** — MCP server that exposes the tool (`mcpTrigger` -> `toolWorkflow`)

**Why two workflows?** n8n's API ignores `specifyInputSchema` when creating workflows via API. The `toolWorkflow` + sub-workflow pattern avoids this bug — parameters arrive via `$json.param` which always works.

The Library Manager imports `sub` first, gets its ID, patches `REPLACE_SUB_WORKFLOW_ID` in `server`, then imports `server`.

---

## Valid Categories

Use one of these existing categories. If none fit, open an issue to propose a new one.

`analytics` · `cms` · `communication` · `creativity` · `e-commerce` · `entertainment` · `finance` · `knowledge` · `language` · `maps` · `marketing` · `meetings` · `network` · `news` · `productivity` · `reference` · `smart-home` · `tourism` · `transport` · `utilities` · `weather`

---

## Naming Conventions

| What | Convention | Example |
|------|-----------|---------|
| Template ID | `lowercase-hyphens` | `weather-openmeteo` |
| Template directory | Same as ID | `templates/weather-openmeteo/` |
| Server workflow name | `MCP: Display Name` | `MCP: Weather OpenMeteo` |
| Sub-workflow name | `MCP Sub: Display Name` | `MCP Sub: Weather OpenMeteo` |
| Tool names | `snake_case` | `get_forecast`, `list_contacts` |
| Credential keys | `lowercase_underscore` | `seafile_token`, `lexware_api_token` |

**Consistency rule:** The template ID must match across: directory name, `manifest.json` `id`, `index.json` `id`, and the MCP trigger `path`. Tool names must match across: `manifest.json` `tools[].name`, the toolWorkflow node `name` field, and the `connections` key.

---

## Critical Rules

### HTTP Requests
```javascript
// CORRECT — use helpers (no $)
const data = await helpers.httpRequest({ method: 'GET', url: '...' });

// WRONG — $helpers is undefined in Code node v2
const data = await $helpers.httpRequest({ method: 'GET', url: '...' });
```

### Parameter Passing

In the **sub-workflow** Code node, parameters arrive via `$input`:
```javascript
const input = $input.first().json;
const city = input.city || 'Berlin';
```

In the **server workflow**, parameters are extracted from the user's message via `$fromAI()`:
```javascript
"city": "={{ $fromAI('city', 'City name', 'string') }}"
```

### Required vs Optional Parameters

**Problem:** When a parameter is marked as `"required": true` in the toolWorkflow schema, the MCP server exposes it as required in `tools/list`. If the AI agent doesn't provide it, the tool call fails — even if the sub-workflow code handles missing values with defaults.

**Rule:** Only mark a parameter as `"required": true` if the tool genuinely cannot work without it. Parameters with defaults in the Code node should be `"required": false`.

**Example — wrong:**
```json
"schema": [
  { "id": "action", "type": "string", "required": true },
  { "id": "limit", "type": "string", "required": true }
]
```
Here `limit` has a default in the Code node (`const lim = parseInt(input.limit) || 10`), so marking it required forces the AI to always provide it — even when the default is fine.

**Example — correct:**
```json
"schema": [
  { "id": "action", "type": "string", "required": true },
  { "id": "limit", "type": "string", "required": false }
]
```

**Safety net:** The MCP Client in n8n-claw auto-fills missing required params with empty strings via `tools/list` schema inspection. But this is a fallback — templates should set `required` correctly in the first place.

### Credential Fetching Pattern

Templates that need API keys fetch them at runtime from `template_credentials` via PostgREST. The placeholders `{{SUPABASE_URL}}` and `{{SUPABASE_SERVICE_KEY}}` are replaced automatically by the Library Manager during install.

**Single credential:**
```javascript
const SUPABASE_URL = '{{SUPABASE_URL}}';
const SUPABASE_KEY = '{{SUPABASE_SERVICE_KEY}}';
const pgHeaders = { 'apikey': SUPABASE_KEY, 'Authorization': 'Bearer ' + SUPABASE_KEY };

const creds = await helpers.httpRequest({
  method: 'GET',
  url: SUPABASE_URL + '/rest/v1/template_credentials?template_id=eq.my-template&cred_key=eq.my_api_key&select=cred_value',
  headers: pgHeaders
});
if (!creds || !creds[0] || !creds[0].cred_value) {
  return [{ json: { error: 'API key not configured. Use add_credential to set it up.' } }];
}
const apiKey = creds[0].cred_value;
```

**Multiple credentials (single query):**
```javascript
const allCreds = await helpers.httpRequest({
  method: 'GET',
  url: SUPABASE_URL + '/rest/v1/template_credentials?template_id=eq.seafile&select=cred_key,cred_value',
  headers: pgHeaders
});
const creds = {};
for (const c of (allCreds || [])) creds[c.cred_key] = c.cred_value;

const url = creds.seafile_url;
const token = creds.seafile_token;
```

### Multi-Tool Templates (Action Routing)

Templates with multiple tools use a single Code node that routes by `action` parameter:

**Sub-workflow (one Code node for all actions):**
```javascript
const input = $input.first().json;
const action = input.action || 'list_items';

if (action === 'list_items') {
  // ...
} else if (action === 'get_item') {
  // ...
} else {
  return [{ json: { error: 'Unknown action: "' + action + '". Available: list_items, get_item.' } }];
}
```

**Server workflow:** One `toolWorkflow` node per tool. Each passes `"action"` as a hardcoded value (not from `$fromAI`):
```json
"workflowInputs": {
  "value": {
    "action": "list_items",
    "query": "={{ $fromAI('query', 'Search query', 'string') }}"
  }
}
```

All toolWorkflow nodes connect to the same MCP Server Trigger and use the same `REPLACE_SUB_WORKFLOW_ID`:
```json
"connections": {
  "list_items": { "ai_tool": [[{ "node": "MCP Server Trigger", "type": "ai_tool", "index": 0 }]] },
  "get_item":   { "ai_tool": [[{ "node": "MCP Server Trigger", "type": "ai_tool", "index": 0 }]] }
}
```

See `templates/seafile/` or `templates/lexware/` for real examples.

### OAuth2 Templates (Google Services)

For Google APIs, templates share a credential pool via `shared_id`:

**manifest.json:**
```json
{
  "credential_type": "oauth2",
  "oauth_config": {
    "provider": "google",
    "shared_credential_id": "google-oauth",
    "scopes": ["https://www.googleapis.com/auth/gmail.readonly"]
  },
  "credentials_required": [
    { "key": "client_id", "label": "Google OAuth Client ID", "shared_id": "google-oauth" },
    { "key": "client_secret", "label": "Google OAuth Client Secret", "shared_id": "google-oauth" }
  ]
}
```

Multiple Google templates (Gmail, Calendar, Drive, etc.) share the same `google-oauth` credential pool. Once one is authorized, others reuse the same tokens.

The Code node needs a `getGoogleToken()` helper that fetches, checks expiry, and refreshes tokens. Copy this function from `templates/gmail/workflow.json` — it is identical across all Google OAuth templates.

### File Bridge

`http://file-bridge:3200` is a Docker-internal service for binary file handling (uploads/downloads).

**Two key endpoints:**
- `POST /upload/base64` — Store a file: `{ content_base64, file_name, mime_type }` -> returns `{ id }` (the `file_ref`)
- `POST /files/{id}/forward` — Forward a stored file to an external API as multipart: `{ url, headers, form_fields, filename }`

Use File Bridge when your template needs to upload user-sent files to cloud storage or download binary files (PDFs, images) to send back to the user.

See `templates/seafile/`, `templates/nextcloud-files/`, or `templates/lexware/` for real examples.

### Error Handling Best Practices

1. **Descriptive error messages with examples:**
```javascript
return [{ json: { error: 'Parameter "query" is required. Use Gmail search syntax (e.g. "from:user@example.com", "is:unread").' } }];
```

2. **Try-catch around every API call:**
```javascript
try {
  const result = await helpers.httpRequest({ method: 'GET', url: apiUrl, headers });
} catch(e) {
  return [{ json: { error: 'Failed to fetch items: ' + e.message } }];
}
```

3. **Truncate large responses** to prevent LLM context from blowing up:
```javascript
if (text.length > 5000) {
  text = text.substring(0, 5000) + '\n\n... (truncated, ' + text.length + ' chars total)';
}
```

### Placeholders

- `REPLACE_SUB_WORKFLOW_ID` — patched automatically by Library Manager during install
- `{{SUPABASE_URL}}` and `{{SUPABASE_SERVICE_KEY}}` — replaced during install (for credential fetching)
- The `path` in mcpTrigger should match the template ID

### Connections

The toolWorkflow connects to mcpTrigger via `ai_tool`, not `main`:
```json
"connections": {
  "tool_name": {
    "ai_tool": [[{ "node": "MCP Server Trigger", "type": "ai_tool", "index": 0 }]]
  }
}
```

---

## index.json Entry Format

**Native template (API key):**
```json
{
  "id": "my-template",
  "name": "My Template",
  "type": "native",
  "category": "utilities",
  "description": "Short description of what this template does",
  "credentials_required": [
    { "key": "my_api_key", "label": "My API Key" }
  ],
  "version": "1.0.0"
}
```

**OAuth2 template (Google):**
```json
{
  "id": "my-google-service",
  "name": "My Google Service",
  "type": "native",
  "category": "productivity",
  "description": "Short description",
  "credential_type": "oauth2",
  "credentials_required": [
    { "key": "client_id", "label": "Google OAuth Client ID", "shared_id": "google-oauth" },
    { "key": "client_secret", "label": "Google OAuth Client Secret", "shared_id": "google-oauth" }
  ],
  "version": "1.0.0"
}
```

Key difference: OAuth2 entries add `"credential_type": "oauth2"` and credentials use `"shared_id"`.

---

## Checklist for New Templates

- [ ] Template ID is lowercase with hyphens only
- [ ] Category is one of the 19 valid categories listed above
- [ ] `manifest.json` has all required fields (see `TEMPLATE_EXAMPLE.md`)
- [ ] `workflow.json` uses format `n8n-claw-template` with `sub` and `server` keys
- [ ] Code uses `helpers.httpRequest()` (NOT `$helpers`)
- [ ] `REPLACE_SUB_WORKFLOW_ID` used as workflowId placeholder
- [ ] Credential fetching uses `{{SUPABASE_URL}}` and `{{SUPABASE_SERVICE_KEY}}` placeholders
- [ ] `index.json` entry added (only your entry — don't touch others)
- [ ] `README.md` skills table updated (correct category, skill count, total count)
- [ ] Tool names use `snake_case` and match across manifest, node names, and connections
- [ ] Workflow names follow `MCP: Name` / `MCP Sub: Name` convention
- [ ] Error messages are descriptive with parameter examples
- [ ] Large responses truncated to ~5000 chars
- [ ] All text in English

---

## Testing

1. Validate JSON: `python -c "import json; json.load(open('templates/my-template/manifest.json')); json.load(open('templates/my-template/workflow.json')); print('OK')"`
2. Check tool name consistency: tool names in `manifest.json` `tools[]`, toolWorkflow node `name` fields, action routing in Code node, and `connections` keys must all match
3. **Local test:** Import sub-workflow into n8n (API or UI), note the ID. Replace `REPLACE_SUB_WORKFLOW_ID` in server JSON with real ID, import server workflow. Test via agent chat or MCP client
4. **Fork test:** Push to a fork, temporarily point the Library Manager's `CDN_BASE` to your fork's raw URL, then install via `install_template`

---

## PR / Contributing Workflow

### Steps

1. Fork the repository
2. Create `templates/{your-id}/manifest.json` and `workflow.json`
3. **`templates/index.json`**: Add your entry at the end of the `templates` array
   - Only add your own entry — do NOT reformat or modify existing entries
4. **`README.md`**: Update the skills table
   - Add your skill to the correct category section (alphabetical order)
   - Update the skill count in the category overview table (e.g. Finance: 3 -> 4)
   - Update the total in the header: "Available Skills (N)"
5. Submit a pull request with a description of what the skill does, which tools it provides, and what credentials are needed

### What NOT to change

- Other templates' files (manifest.json, workflow.json)
- Existing entries in `index.json` (no reformatting, no version bumps)
- Other templates' JSON formatting (even if inconsistent)

### Versioning

- New templates: `1.0.0`
- Bug fixes: bump patch (`1.0.1`)
- New tools/features: bump minor (`1.1.0`)
- Breaking changes (renamed tools, changed parameters): bump major (`2.0.0`)

---

## Reference Files

- `templates/TEMPLATE_EXAMPLE.md` — Annotated example with all manifest and workflow fields explained
- `templates/weather-openmeteo/` — Simple template, no credentials, single tool
- `templates/news-newsapi/` — Simple credential-based template (API key pattern)
- `templates/seafile/` — Multi-tool template with File Bridge (complex example)
- `templates/gmail/` — OAuth2 template with Google token refresh (reference for Google services)
- `templates/lexware/` — Large multi-tool template with 23 tools (most comprehensive example)

---
> Source: [freddy-schuetz/n8n-claw-templates](https://github.com/freddy-schuetz/n8n-claw-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
