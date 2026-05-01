## llm-assisted-project-template

> All SQLite database files should be stored in the `db` directory at the root of the project. This includes:

# Database Storage Rule

All SQLite database files should be stored in the `db` directory at the root of the project. This includes:
- Database files (*.db)
- Database-related configuration files
- Database migration files

The path structure should be:
```
/db/
  ├── databases/
  │   ├── database1.db
  │   ├── database2.db
  │   └── ...
  ├── migrations/
  │   └── ...
  └── config/
      └── ...
```

This rule ensures consistent database file storage across the project and prevents database files from being scattered in different locations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lnsy-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
