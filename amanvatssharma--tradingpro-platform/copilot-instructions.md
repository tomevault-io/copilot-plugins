## tradingpro-platform

> - Always generate clean, readable, enterprise-grade code.


# STYLE
- Always generate clean, readable, enterprise-grade code.
- Prefer explicit imports over wildcard imports.
- Use async/await only, never .then() chaining.
- Keep functions small & composable (max 40 lines).
- Prefer dependency injection over direct instantiation.
- Update changelog in module doc after every edit.

# COMMENTS & DOCS
- Always add detailed file header at top.
- Inline comments where logic may confuse a new developer.
- Auto-update MODULE_DOC.md after edits.
- Generate flowcharts for complex flows using Mermaid or Draw.io.

# LOGGING & DEBUG
- Always inject logger instead of console.log.
- For debugging, add logger.debug at start/end of service methods.
- Mark all TODOs with `[SonuRamTODO]` for easy search.

# STRUCTURE
- Keep controllers thin → delegate to services.
- Services handle business logic only.
- Repositories handle DB logic only.
- No circular dependencies (check with madge).

# API
- For external APIs: always wrap in provider class.
- For internal APIs: choose REST vs GraphQL based on:
  - REST for transactional endpoints (orders, auth, funds).
  - GraphQL for dashboards & data aggregation (positions, reporting).

# CODING
- Use TypeScript strict mode.
- Enforce consistent return types.
- Always export interfaces from module index.ts.

# INTERACTION WITH CURSOR
- Always confirm the plan with me before proceeding.
- Ask me if multiple solutions exist → then suggest the best.
- Generate docs + tests along with code in same PR.
- Regularly scan for duplicate files and unused imports.

# PERSONAL
- Respect Hindu date-time format (IST) in logs.
- Keep naming spiritual/inspiring if possible (like `dharmaGuard` for auth middleware).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AmanVatsSharma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
