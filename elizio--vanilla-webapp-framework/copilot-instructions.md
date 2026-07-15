## documentation

> README how-to maintenance and in-code documentation standards

## **5. Documentation Rules**

See [AGENTS.md](../AGENTS.md) for narrative policy. These rules are enforceable minimums.

### **README how-to maintenance (Rule 5.1)**
- **Rule 5.1a**: `README.md` is the canonical human how-to. Keep these categories accurate and runnable:
  1. **Project setup** — install, `.env` / `.env.example`, database init (`init_db` vs Alembic).
  2. **Run by environment** — development (dual server), production, Docker, testing.
  3. **Configure & operate** — OAuth redirects, **billing provider dashboards/webhooks** (see `docs/billing-configuration.md`), logs, Swagger, migrations, troubleshooting.
- **Rule 5.1b**: Before claiming work done, scan your diff against README sync triggers in `AGENTS.md` → Documentation policy. Update matching how-to sections and `.env.example` when env vars change. Multi-step provider setup (OAuth, billing) belongs in `docs/oauth-configuration.md` or `docs/billing-configuration.md` — link from README, do not duplicate steps.
- **Rule 5.1c**: Do not copy AGENTS recipes into README. Do not duplicate README prose in AGENTS.

### **In-code documentation (Rule 5.2)**
- **Rule 5.2a**: Every Python module (`backend/**/*.py`) starts with a one-line module docstring.
- **Rule 5.2b**: Public Python functions and classes use Google-style docstrings and type hints on signatures.
- **Rule 5.2c**: Every API route keeps its Flasgger Swagger YAML block (`---`); do not duplicate it in a separate docstring.
- **Rule 5.2d**: Exported JavaScript modules (`frontend/src/**/*.js`) include a file-level JSDoc block; exported functions/objects include `@param` / `@returns` when not self-evident.
- **Rule 5.2e**: Document new and modified code only — add module docs and document public symbols you touch in files you edit. No repo-wide retroactive pass.

### **Anti-patterns (Rule 5.3)**
- Restating the function name in a docstring.
- README-style essays in module docstrings.
- Claiming unimplemented features in README (e.g. DigitalOcean deploy until built).

---
> Source: [Elizio/vanilla-webapp-framework](https://github.com/Elizio/vanilla-webapp-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
