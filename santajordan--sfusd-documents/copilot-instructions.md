## sfusd-documents

> Raw materials and analysis outputs for SFUSD spending, labor agreements, and the 2026 UESF strike.

# SFUSD District Document Library

Raw materials and analysis outputs for SFUSD spending, labor agreements, and the 2026 UESF strike.

## Supplemental Contract Archive

The `analysis/contracts/` directory is **git-ignored** (3.0 GB, ~1,181 PDFs). It contains vendor contracts from 64 SFUSD contracts and 180 peer California school districts, used for cross-district vendor payment comparison.

**You must download it before working with contract data:**

```bash
pip install gdown
gdown "https://drive.google.com/uc?id=1fSnfhrYENywoUJ9dKh3x-BEutuC4s6pZ" -O analysis/contracts.zip
unzip analysis/contracts.zip -d analysis/contracts/
rm analysis/contracts.zip
```

## File Naming Convention

All files follow: `{source}_{description}_{date-or-fy}.{ext}`

Examples:
- `sfusd_adopted-budget-sacs-2nd-reading_fy2025-26.pdf`
- `perb_decision-2906e-oakland-usd_2024.pdf`
- `sf-chronicle_admin-spending-audit_2023-01-10.txt`

Text files (`.txt`) include a source URL header at the top.

## Full Inventory

See `README.md` for complete directory structure, file counts, and descriptions of every document category.

---
> Source: [SantaJordan/sfusd-documents](https://github.com/SantaJordan/sfusd-documents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
