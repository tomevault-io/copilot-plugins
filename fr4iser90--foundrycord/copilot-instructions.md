## foundrycord

> - Overall System Overview: @docs/3_developer_guides/02_architecture/overview.md

# Backend/Shared/Bot Development Context & Rules

## Key Architecture Documents

- Overall System Overview: @docs/3_developer_guides/02_architecture/overview.md
- Backend (FastAPI) Design: @docs/3_developer_guides/02_architecture/backend_design.md
- Database Schema: @docs/3_developer_guides/02_architecture/database_schema.md
- API Specification: @docs/3_developer_guides/02_architecture/api_specification.md
- Relevant Coding Conventions (Python, DB): @docs/3_developer_guides/01_getting_started/coding_conventions.md
- Technology Stack (Backend/Bot/DB Sections): @docs/1_introduction/tech_stack.md

## Code Verification Principles

- **Imports:** Briefly check the definition/signature of imported functions/classes before use.
- **API/Service Calls:** Verify expected parameters and responses of internal services or external APIs (e.g., Discord) based on documentation or code.
- **Config/Data Access:** Anticipate that config values or DB data might be missing and handle robustly (e.g., defaults, error handling).

*Note: Critical core rules (Instruction Adherence, etc.) are defined in `critical_rules.mdc` and apply always.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fr4iser90)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fr4iser90)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
