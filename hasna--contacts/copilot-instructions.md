## contacts

> This guide explains how to use the `@hasna/contacts` MCP server effectively as an AI coding agent.

# AGENTS.md — Guide for AI Agents

This guide explains how to use the `@hasna/contacts` MCP server effectively as an AI coding agent.

## Setup

```bash
# Install globally
bun install -g @hasna/contacts

# Register with Claude Code
claude mcp add --transport stdio --scope user contacts -- contacts-mcp
```

After installing, restart Claude Code and verify with `/mcp`.

## Core Workflow

### Finding contacts

Start with search — don't list everything:

```
search_contacts({ query: "John" })
search_contacts({ query: "acme corp" })
list_contacts({ tag: "client", limit: 20 })
```

### Creating a contact

Minimum required: at least one of `first_name`, `last_name`, or `company_id`.

```
create_contact({
  first_name: "Jane",
  last_name: "Doe",
  email: "jane@example.com",
  phone: "+1-555-0100",
  notes: "Met at conference 2026"
})
```

### Adding structure

Use tags for grouping, relationships for linking people:

```
create_tag({ name: "client", color: "#3b82f6" })
add_tag_to_contact({ contact_id: "...", tag_id: "..." })
add_relationship({ contact_a_id: "...", contact_b_id: "...", relationship_type: "colleague" })
```

### Bulk operations

Import from CSV or vCard when adding many contacts:

```
import_contacts({ format: "csv", data: "first_name,last_name,email\nJane,Doe,jane@example.com" })
import_contacts({ format: "vcf", data: "BEGIN:VCARD\n..." })
```

Export for backup or sharing:

```
export_contacts({ format: "json" })
export_contacts({ format: "vcf", tag: "client" })
```

## All MCP Tools

| Tool | Key Parameters | Returns |
|------|----------------|---------|
| `create_contact` | first_name, last_name, email, phone, company_id, notes | Contact object |
| `get_contact` | id | Contact with all details |
| `update_contact` | id, ...fields | Updated contact |
| `delete_contact` | id | Success message |
| `list_contacts` | tag, company_id, limit, offset, order_by | Contact list |
| `search_contacts` | query, limit | Ranked contact list |
| `create_company` | name, website, industry, notes | Company object |
| `get_company` | id | Company with contacts |
| `update_company` | id, ...fields | Updated company |
| `delete_company` | id | Success message |
| `list_companies` | limit, offset | Company list |
| `search_companies` | query | Ranked company list |
| `create_tag` | name, color | Tag object |
| `list_tags` | — | All tags with counts |
| `add_tag_to_contact` | contact_id, tag_id | Success message |
| `remove_tag_from_contact` | contact_id, tag_id | Success message |
| `add_relationship` | contact_a_id, contact_b_id, relationship_type | Relationship object |
| `list_relationships` | contact_id | All relationships for contact |
| `merge_contacts` | source_id, target_id | Merged contact |
| `import_contacts` | format, data | Import summary |
| `export_contacts` | format, tag, company_id | Exported string |
| `get_stats` | — | DB stats (counts, etc.) |
| `get_activity` | contact_id, limit | Activity log |
| `log_activity` | contact_id, type, notes | Activity entry |

## Database Location

By default: `~/.contacts/contacts.db`

Override with env var: `CONTACTS_DB_PATH=/path/to/contacts.db contacts-mcp`

## Tips for Agents

- **Always search before creating** — avoid duplicate contacts
- **Use `get_stats`** at session start to understand the current database state
- **Use notes generously** — add context about how/when you met someone
- **Tag consistently** — agree on a tag taxonomy early (e.g., `client`, `lead`, `colleague`, `vip`)
- **Merge duplicates** when found — `merge_contacts` preserves all data from both contacts
- **Log activities** for important interactions — `log_activity` keeps a timeline per contact

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hasna)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hasna)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
