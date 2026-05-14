## migration-file-creation

> When generating migration files, use the command `date +%s000` to get the timestamp for the prefix of the migration file.


When generating migration files, use the command `date +%s000` to get the timestamp for the prefix of the migration file.

There must be tow parts in the sql file: the up migration and the down migration.

The up migration should be after the comment `-- Up Migration` and the down migration should be after the comment `-- Down Migration`.

---
> Source: [nicnocquee/dataqueue](https://github.com/nicnocquee/dataqueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
