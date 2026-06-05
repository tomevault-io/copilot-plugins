## backend-db-migrations

> Backend SQL migration path and validation script


# Backend Database Migrations

- Migrations live in `qubership-apihub-service/resources/migrations/`.
- Use the next unused numeric prefix (current highest is visible in that directory).
- **Never** reuse or duplicate migration numbers.
- Provide paired `.up.sql` and `.down.sql` files when rollback is required.
- After adding migrations, run from repo root:
  - Linux / WSL / Git Bash: `bash .cursor/skills/apihub-backend-developer/scripts/check_migration_numbers.sh`
  - Windows PowerShell: `powershell -File .cursor/skills/apihub-backend-developer/scripts/check_migration_numbers.ps1`

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
