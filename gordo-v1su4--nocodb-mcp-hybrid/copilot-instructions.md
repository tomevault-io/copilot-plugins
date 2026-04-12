## nocodb-mcp-hybrid

> Guidance for writing correct n8n expressions ({{ }} syntax). Use when mapping data between nodes or accessing $json, $node, or $now variables.


# n8n Expression Syntax

Expert guide for writing correct n8n expressions in workflows.

## The Rule of Double Braces
All dynamic content in n8n MUST be wrapped in double curly braces:
- ✅ `{{$json.email}}`
- ✅ `{{$json.body.name}}`
- ✅ `{{$node["HTTP Request"].json.data}}`
- ❌ `$json.email` (literal text)
- ❌ `{{$json.email}}` (inside a Code node)

## 🚨 CRITICAL: Webhook Data Structure
**Webhook data is NEVER at the root.** It is always nested under `.body`.
- ❌ WRONG: `{{$json.name}}`
- ✅ CORRECT: `{{$json.body.name}}`

## Referencing Other Nodes
- Node names MUST be in quotes and are case-sensitive.
- ✅ `{{$node["HTTP Request"].json.data}}`
- ❌ `{{$node.HTTP Request.json.data}}` (Missing quotes)
- ❌ `{{$node["http request"].json.data}}` (Wrong case)

## Core Variables
- `$json`: Current node output.
- `$node["Name"]`: Reference a specific node.
- `$now`: Current timestamp (Luxon). Use `{{$now.toFormat('yyyy-MM-dd')}}`.
- `$env`: Environment variables.

## Expressions vs. Code Node
- **Other Nodes**: Use `{{$json.field}}`.
- **Code Nodes**: Use direct JavaScript access `$json.field` or `$input.item.json.field`. **DO NOT** use `{{ }}` in Code nodes.

## Common Fixes
- **Spaces in fields**: Use `{{$json['field name']}}`.
- **Default values**: Use `{{$json.field || 'default'}}`.
- **Simple Logic**: Use ternary `{{$json.status === 'active' ? 'Yes' : 'No'}}`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gordo-v1su4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gordo-v1su4)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
