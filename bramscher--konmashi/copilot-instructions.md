## konmashi

> - Project ID: `ovzistntcsomqddezegq`

# Database Safety Rules

## Supabase Project Configuration
- Project ID: `ovzistntcsomqddezegq`
- Project Name: konmashi-start
- Region: us-east-1

## Critical Safety Protocols

### Data Protection
- NEVER run destructive database commands without explicit user confirmation
- ALWAYS check for backups before schema changes
- NEVER assume user wants to lose data
- ALWAYS use the correct project ID for Supabase MCP operations

### Database Operations Safety
1. Schema Changes
   - Create migrations incrementally
   - Test schema changes in development first
   - Document all database changes
   - Preserve existing data during migrations

2. Data Operations
   - Backup sensitive data before modifications
   - Use transactions for critical operations
   - Validate data before bulk operations
   - Keep audit logs of significant changes

3. Access Control
   - Maintain proper RLS policies
   - Review permission changes carefully
   - Monitor access patterns
   - Regular security audits

Remember: **Data loss is never acceptable unless explicitly requested and confirmed by the user.**

## CRITICAL DATABASE SAFETY PROTOCOLS

### 🚫 NEVER RUN THESE COMMANDS WITHOUT EXPLICIT USER CONFIRMATION:

1. **SQL DROP commands** (`DROP TABLE`, `DROP COLUMN`, `DROP DATABASE`)
   - These permanently delete database objects and data
   - Always ask: "This will DELETE DATA. Are you sure?"

2. **`TRUNCATE TABLE`** - DESTROYS ALL DATA IN TABLE
   - Removes all rows from a table
   - Require explicit confirmation before running

3. **`DELETE FROM table`** without WHERE clause
   - Can delete all rows if WHERE clause is missing
   - Always verify WHERE conditions

4. **ALTER TABLE operations that remove columns**
   - `ALTER TABLE ... DROP COLUMN` permanently removes data
   - Require explicit confirmation

### ✅ CORRECT DATABASE OPERATION WORKFLOW:

**This project uses Supabase MCP tools, NOT Prisma:**

1. **For Schema Changes:**
   - Use `mcp_user-supabase_apply_migration` to create and apply migrations
   - Example: Adding columns, creating tables, modifying constraints
   - Migrations are versioned and tracked in Supabase

2. **For Data Queries:**
   - Use `mcp_user-supabase_execute_sql` for SELECT, INSERT, UPDATE queries
   - Do NOT use for DDL operations (use apply_migration instead)

3. **For Schema Inspection:**
   - Use `mcp_user-supabase_list_tables` to see table structure
   - Use `mcp_user-supabase_execute_sql` with SELECT queries to inspect data

### ✅ SAFE DATABASE MIGRATION WORKFLOW:

**This project uses Supabase MCP, NOT Prisma:**

1. **For Schema Changes:**
   - Use `mcp_user-supabase_apply_migration` with:
     - `project_id`: The Supabase project ID
     - `name`: Descriptive migration name (snake_case)
     - `query`: SQL DDL statements (ALTER TABLE, CREATE TABLE, etc.)
   - Example: Adding a column, creating a table, modifying constraints

2. **For Data Queries:**
   - Use `mcp_user-supabase_execute_sql` for:
     - SELECT queries to inspect data
     - INSERT/UPDATE/DELETE operations (with caution)
   - Do NOT use for DDL operations

3. **For Schema Inspection:**
   - Use `mcp_user-supabase_list_tables` to see all tables
   - Use `mcp_user-supabase_list_migrations` to see migration history

### 🔍 REQUIRED SAFETY CHECKS:

Before any database command:
1. **Ask about backups:** "Do you have recent database backups?"
2. **Confirm environment:** "Is this development or production?"
3. **Explain consequences:** Clearly state what data will be affected
4. **Wait for explicit approval:** Never assume permission for destructive operations

### 📋 DATABASE OPERATION CHECKLIST:

- [ ] Backup exists and is recent
- [ ] User understands the consequences
- [ ] Environment is confirmed (dev/staging/prod)
- [ ] Non-destructive alternatives considered
- [ ] User has given explicit written approval

### 🛡️ EMERGENCY RECOVERY:

If destructive command was run accidentally:
1. **STOP immediately** - don't run additional commands
2. **Check backup status** with user
3. **Guide backup restoration** process
4. **Document what happened** for future prevention

### 💾 BACKUP VERIFICATION:

Before any schema changes:
- Confirm backup recency (within 24 hours for prod)
- Verify backup integrity if possible
- Document backup restoration process

## EXAMPLES OF SAFE vs UNSAFE:

### ❌ UNSAFE (Never do without explicit confirmation):
```sql
DROP TABLE table_name;
DROP COLUMN column_name;
TRUNCATE TABLE table_name;
DELETE FROM table_name; -- without WHERE clause
ALTER TABLE ... DROP COLUMN ...;
```

### ✅ SAFE (Preferred approaches):
```typescript
// Use Supabase MCP for schema changes
mcp_user-supabase_apply_migration({
  project_id: "your-project-id",
  name: "add_column_name",
  query: "ALTER TABLE table_name ADD COLUMN column_name TYPE;"
})

// Use Supabase MCP for data queries
mcp_user-supabase_execute_sql({
  project_id: "your-project-id",
  query: "SELECT * FROM table_name WHERE condition;"
})
```

## USER COMMUNICATION:

Always use phrases like:
- "This command will DELETE ALL DATA. Do you want to proceed?"
- "Let me suggest a safer approach..."
- "Before we make schema changes, do you have recent backups?"
- "This is a destructive operation. Please confirm you want to continue."

## NEVER ASSUME:

- That the user wants to lose data
- That "development" means data doesn't matter  
- That the user understands the consequences
- That previous permission applies to new destructive commands

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bramscher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
