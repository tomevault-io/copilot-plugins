## fabric-scd2-customer-history

> This repository demonstrates a Microsoft Fabric SCD Type 2 Customer Dimension solution. Keep changes aligned with that portfolio objective and avoid unrelated tooling or infrastructure.

# Repository Guidance

## Scope

This repository demonstrates a Microsoft Fabric SCD Type 2 Customer Dimension solution. Keep changes aligned with that portfolio objective and avoid unrelated tooling or infrastructure.

## Working Agreements

- Preserve the existing repository structure unless a documented design change requires otherwise.
- Keep representative data synthetic and free of personal, confidential, or production information.
- Document assumptions, dependencies, and validation steps alongside implementation changes.
- Keep SQL scripts focused, readable, and independently reviewable when SQL development begins.
- Store Fabric-related definitions and deployment guidance under `fabric/`.
- Store Power BI assets and supporting documentation under `power-bi/`.
- Do not commit credentials, connection strings, access tokens, local settings, or generated temporary files.
- Update the root README when project status, architecture, or setup expectations change.

## Validation

Changes should include proportionate validation. Data-processing work should cover initial loads, unchanged records, attribute changes, new customers, and repeat processing where applicable.

---
> Source: [DJ8052/fabric-scd2-customer-history](https://github.com/DJ8052/fabric-scd2-customer-history) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
