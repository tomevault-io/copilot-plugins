## go-migrations

> SQL migration numbering and file conventions


# Database Migrations

- Use the next unused numeric prefix (check the migrations directory for the current highest).
- **Never** reuse or duplicate migration numbers.
- Provide paired `.up.sql` and `.down.sql` files when rollback is required.
- After adding migrations, run the repository's migration validation script if one is provided (see the repo-specific developer skill).

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
