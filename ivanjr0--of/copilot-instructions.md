## of

> rebuild: docker compose down && docker compose up --build -d

# Cursor Rules

## Rebuild and restart all services
rebuild: docker compose down && docker compose up --build -d

## Python Import Rules
- Always place all imports at the top of the file
- No imports inside functions or methods  
- Use absolute imports for internal code (e.g., `from app.services import something` instead of `from services import something`)
- Group imports in the following order:
  1. Standard library imports
  2. Third-party imports
  3. Local application imports
- Separate each group with a blank line

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivanjr0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
