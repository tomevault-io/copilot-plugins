## avoid-using-dboperations-table

> When interacting with the database, do not try to add a new method to the DbOperations class. This is because the DbOperations class is deprectaed.

When interacting with the database, do not try to add a new method to the DbOperations class. This is because the DbOperations class is deprectaed.

Instead, the functions / methods in the service layer should perform database requests directly.

---
> Source: [Government-Communication-Service/assist_service](https://github.com/Government-Communication-Service/assist_service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
