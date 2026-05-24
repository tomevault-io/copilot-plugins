## simt-flows-db

> The authoritative source for the database schema is defined in the following file:

# AI Agent Instructions

## Database Schema

The authoritative source for the database schema is defined in the following file:

- `sql/flows-schema-mgf.sql`

When you need to understand table structures, build queries, or identify relationships between tables, **always** refer to this SQL file first. This is significantly faster and more reliable than querying database catalogs or table metadata directly.

### Key Guidelines

- **Table Structure:** Read `sql/flows-schema-mgf.sql` to find `CREATE TABLE` statements.
- **Constraints:** Check for `FOREIGN KEY`, `UNIQUE`, and `NOT NULL` constraints in that file.
- **Indexes:** Look for `CREATE INDEX` statements to understand performance considerations.
- **Views/Functions:** If defined, they will also be located in this or related SQL files in the `sql/` directory.

Do not attempt to introspect the live database unless specifically asked to perform a data migration or debug a runtime state that cannot be inferred from the schema definition.

---
> Source: [cepro/simt-flows-db](https://github.com/cepro/simt-flows-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
