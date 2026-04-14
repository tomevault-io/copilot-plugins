## thelostandunfounds

> **NEVER put DDL (Data Definition Language) statements inside a DO $$ block.**

# SQL Syntax Validation Rules

## CRITICAL: DDL Statements Cannot Be Inside DO Blocks

**NEVER put DDL (Data Definition Language) statements inside a DO $$ block.**

### What is DDL?
DDL statements define database structure:
- `CREATE TABLE`
- `CREATE INDEX`
- `CREATE FUNCTION`
- `CREATE POLICY`
- `ALTER TABLE`
- `DROP TABLE`
- `DROP POLICY`
- `GRANT`
- `REVOKE`

### What CAN be inside DO blocks?
Only DML (Data Manipulation Language) and control flow:
- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`
- `IF/THEN/ELSE`
- `LOOP`
- `RAISE NOTICE/WARNING/EXCEPTION`

### Correct Pattern

```sql
-- ✅ CORRECT: DDL outside DO block
CREATE TABLE IF NOT EXISTS my_table (...);
CREATE INDEX IF NOT EXISTS idx_name ON my_table(column);

-- ✅ CORRECT: DML inside DO block
DO $$
DECLARE
  var_name TYPE;
BEGIN
  SELECT id INTO var_name FROM table WHERE ...;
  IF var_name IS NOT NULL THEN
    INSERT INTO table VALUES (...);
  END IF;
END $$;
```

### Incorrect Pattern

```sql
-- ❌ WRONG: DDL inside DO block
DO $$
BEGIN
  CREATE TABLE my_table (...);  -- ERROR: syntax error at or near "BEGIN"
  CREATE INDEX idx_name ON my_table(column);  -- ERROR
END $$;
```

## Validation Checklist

Before committing any SQL file:

1. ✅ **Check for DDL in DO blocks**: Search for `CREATE`, `ALTER`, `DROP`, `GRANT` inside `DO $$` blocks
2. ✅ **Verify function syntax**: Functions must be created outside DO blocks
3. ✅ **Check policy creation**: Policies must be created outside DO blocks
4. ✅ **Verify table creation**: Tables must be created outside DO blocks
5. ✅ **Test with simple query first**: If possible, validate syntax before committing

## Common Errors

1. **Error: syntax error at or near "BEGIN"**
   - Cause: DDL statement inside DO block
   - Fix: Move DDL outside DO block

2. **Error: cannot execute CREATE TABLE in a non-atomic block**
   - Cause: DDL inside transaction block
   - Fix: Move DDL outside transaction or use separate statements

3. **Error: function cannot be created inside DO block**
   - Cause: CREATE FUNCTION inside DO $$
   - Fix: Move CREATE FUNCTION before DO block

## Best Practices

1. **Structure SQL files as:**
   ```sql
   -- 1. Functions first
   CREATE OR REPLACE FUNCTION ...;
   
   -- 2. Tables and indexes
   CREATE TABLE IF NOT EXISTS ...;
   CREATE INDEX IF NOT EXISTS ...;
   
   -- 3. Policies
   CREATE POLICY ...;
   
   -- 4. Data operations in DO block
   DO $$
   BEGIN
     -- Only DML here
   END $$;
   ```

2. **Always test SQL in Supabase SQL Editor before committing**

3. **Use IF NOT EXISTS for idempotent operations**

4. **Document why DDL is outside DO block in comments**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canyouseeus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
