## platform-generator

> MultiDataTable Config

rule MultiDataTableConfig
description "Ensure correct definition of MultiDataTable configurations."
config-file "src/db/MultiDataTable.ts"
checks [
  { tableName: "tb_platforms", fields: ["++id", "name", "*apps", "*specifications"] },
  { tableName: "appsDB", fields: ["++id", "name", "*microservices"] },
  { tableName: "microservicesDB", fields: ["++id", "name", "*databases"] },
  { tableName: "storesDB", fields: ["++id", "name", "*frontends"] }
]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Malnati) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
