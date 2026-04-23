## metamask-mobile-automation

> **NEVER REMOVE OR RESET n8n DOCKER VOLUMES - DATA LOSS INCIDENT PREVENTION**

# CRITICAL n8n DATA PROTECTION RULES
**NEVER REMOVE OR RESET n8n DOCKER VOLUMES - DATA LOSS INCIDENT PREVENTION**

## FORBIDDEN COMMANDS - WILL CAUSE DATA LOSS:
- `docker-compose down -v` (removes volumes)
- `docker volume rm` (removes volumes)  
- `docker system prune -a --volumes` (removes all volumes)
- Any command that removes the n8n volume or container data

## SAFE n8n MANAGEMENT:
- Use `docker-compose restart` to restart services
- Use `docker-compose stop` and `docker-compose up` to stop/start
- ALWAYS backup before any changes: `./backup-n8n.sh`

## MANDATORY BACKUP BEFORE ANY n8n CHANGES:
```bash
# Create backup before ANY workflow modifications
./backup-n8n.sh

# Export individual workflows (manual backup):
# 1. Go to n8n UI → Workflows
# 2. Click workflow → Settings → Export
# 3. Save to workflows/backups/
```

## WORKFLOW RECOVERY:
If data loss occurs:
1. Stop n8n: `docker-compose stop`
2. Restore database: `cp backup/database-*.sqlite to container`
3. Fix permissions: `chown node:node database.sqlite`
4. Restart: `docker-compose up`

**CRITICAL**: The n8n volume contains ALL workflows, credentials, and execution history. Removing it destroys everything and causes user rage.

## n8n API Workflow Update Rules
- **NEVER recreate entire workflows** - causes ID changes and loses connections
- **MCP diff updates often fail** - use direct API calls instead
- **API Key**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJiZjVmYmI3ZS0wZjlmLTQ1ZmEtYjU2NC02NjZkNjEzNTkzNTUiLCJpc3MiOiJuOG4iLCJhdWQiOiJwdWJsaWMtYXBpIiwiaWF0IjoxNzU2Njk5MzAzfQ.JAkCye5OFJGXiBfyzDunBerwxkR8kDxfuYdkvBSW6W0`
- **API Headers**: Use `X-N8N-API-KEY: {token}` not `Authorization: Bearer {token}`
- **PATCH method not allowed** - use PUT for updates
- **Preserve workflow structure** - only update specific node parameters

## How to Edit Individual n8n Nodes via API
Since MCP diff updates fail, use direct API calls to update specific nodes:

1. **Get current workflow structure**: 
   ```bash
   curl -X GET http://localhost:5678/api/v1/workflows/{workflow_id} \
     -H "X-N8N-API-KEY: {api_key}"
   ```

2. **Update workflow with modified node**:
   ```bash
   curl -X PUT http://localhost:5678/api/v1/workflows/{workflow_id} \
     -H "X-N8N-API-KEY: {api_key}" \
     -H "Content-Type: application/json" \
     -d '{"name": "...", "nodes": [...], "connections": {...}}'
   ```

3. **Important restrictions**:
   - Don't include `active` field (read-only)
   - Must include ALL nodes and connections
   - Preserve node IDs and positions
   - Only modify the `parameters.jsCode` for Code nodes

4. **Example script**: See `/scripts/update-scoring-node.sh` for reference

# n8n workflow development rules
**CRITICAL n8n patterns to avoid mistakes:**

## Data Flow Rules
- **NEVER lose context**: Each node MUST pass through ALL previous data plus its new data
- **Pattern**: `return [{ json: { ...previousData, newField: newValue } }]`
- After Execute Command nodes, add Code nodes to merge command output with previous context
- Use `$('NodeName').first()` to access data from specific previous nodes

## Code Node Limitations & Data Access
- **NEVER use `fs` module** - not available in n8n environment
- **NEVER use `require('fs')` or `fs.readFileSync()`** - will always fail
- **NEVER use `items[0].json`** - this variable doesn't exist
- **CORRECT**: Use `$input.first()` and `$input.all()` for current node data
- **CORRECT**: Use `$('Node Name').first()` for other node data
- Use Execute Command nodes with `cat` command instead: `cat /path/to/file`

## Execute Command Node Syntax - CORRECTED
- **n8n expressions**: Use `{{ $json.field }}` NOT `${{ $json.field }}`
- **Shell substitution error**: Remove `$` before `{{` in n8n expressions
- **JSON escaping problem**: Never use `echo` with JSON - use temp files or printf
- Execute Commands only output their own result, losing previous context

## WRONG Execute Command Patterns
```bash
# FAILS - Shell escaping breaks with quotes in JSON
echo '{{ JSON.stringify($json.data) }}' | node script.js

# FAILS - Data access doesn't exist  
echo '{{ items[0].json.field }}' | script.js
```

## CORRECT Execute Command Patterns
```bash
# Method 1: Write to temp file first
printf '%s' '{{ JSON.stringify($json.data) }}' > /tmp/input.json && node script.js --input /tmp/input.json

# Method 2: Simple field access (no JSON)
node script.js {{ $json.bugId }}

# Method 3: Use cat with existing file
cat {{ $json.filePath }} | node script.js

# Method 4: Environment variable approach
BUGID={{ $json.bugId }} node script.js
```

## Merge Node Configuration
- **Mode**: Use "append" or "merge", NEVER "passThrough" (not supported)
- Merge nodes combine inputs but don't automatically preserve all data

## Data Access Patterns - CORRECTED SYNTAX
- **NEVER use `items[0].json`** - this doesn't exist in n8n Code nodes
- Access current node input: `$input.first()` (single item) or `$input.all()` (array)
- Access specific node data: `$('Node Name').first()` or `$('Node Name').all()`
- Pass data between nodes: Always merge previous context with new data

## n8n TypeScript Error Prevention - CRITICAL
- **Node data access**: ALWAYS split into two steps to avoid TypeScript errors:
  ```javascript
  // WRONG - causes TypeScript errors
  const data = $('Node Name').first().json;
  
  // CORRECT - split access
  const nodeData = $('Node Name').first();
  const data = nodeData.json;
  ```
- **Console API**: Only `console.log()` works - replace ALL `console.error()`, `console.warn()`
- **Execute Command wrapping**: Output is wrapped in `.json` - always handle both:
  ```javascript
  const wrapped = $input.first();
  const output = wrapped.json || wrapped; // Handle both cases
  ```
- **Field names**: Execute Command returns `exitCode` NOT `code`

## Correct Code Node Patterns
```javascript
// Access current input data
const currentData = $input.first(); // Single item
const allInputs = $input.all(); // All items as array

// Access data from other nodes
const prevNodeData = $('Previous Node Name').first();
const allPrevData = $('Previous Node Name').all();

// Merge pattern for Execute Command results
const prevContext = $('Previous Node Name').first();
const commandOutput = $input.first();

const merged = {
  ...prevContext,
  newField: commandOutput.stdout || commandOutput.result
};

return merged; // Note: return data directly, not wrapped in array
```

## Shell Command Escaping Rules - CRITICAL
- **NEVER use**: `echo '{{ JSON.stringify($json.data) }}' | script.js`
- **Causes shell escaping failures with quotes in JSON**
- **CORRECT patterns**:
```bash
# Method 1: Use printf instead of echo
printf '%s' {{ JSON.stringify($json.data) }} > /tmp/data.json && script.js < /tmp/data.json

# Method 2: Use environment variables
script.js # with data passed via environment

# Method 3: Use files
script.js --input-file /tmp/input.json
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abretonc7s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
