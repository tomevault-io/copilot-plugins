## corefileawareness

> This rule enforces strict awareness of essential project files. Cursor AI **must always consider, reference, and respect** these files during any form of generation, edits, deployments, or config updates.

# 🧠 Cursor Rule: Core File Awareness (Sesh-Tracker.com)

This rule enforces strict awareness of essential project files. Cursor AI **must always consider, reference, and respect** these files during any form of generation, edits, deployments, or config updates.

---

## ⚙️ 1. Project Configuration

| File                  | Purpose                                                      |
|-----------------------|--------------------------------------------------------------|
| `package.json`        | Declares dependencies, scripts, project metadata             |
| `package-lock.json`   | Ensures consistent dependency resolution                     |
| `tsconfig.json`       | Base TypeScript configuration                                |
| `tsconfig.app.json`   | App-specific TS overrides (if present)                       |
| `tsconfig.node.json`  | Vite + Node-specific configuration                           |
| `tsconfig.worker.json`| Cloudflare Worker-specific TS rules                         |
| `vite.config.ts`      | Vite bundler configuration                                   |
| `postcss.config.js`   | Tailwind/PostCSS plugin configuration                        |
| `eslint.config.js`    | ESLint rules for consistent linting                          |
| `wrangler.json`       | Cloudflare Worker + D1 database configuration                |
| `worker-configuration.d.ts` | Type definitions for Worker environment               |

---

## 🧩 2. Source Entry Points

| File/Folder          | Purpose                                               |
|----------------------|-------------------------------------------------------|
| `src/api/index.ts`   | Cloudflare Worker API entrypoint via Hono            |
| `src/api/routes/`    | Session/inventory endpoint definitions                |
| `src/api/middleware/`| Auth validation, error handling, logging              |
| `src/api/kush-proxy/`| Proxies for Kush.Observer endpoints                   |
| `src/react-app/`     | SPA React application (Vite-based)                    |
| `src/config/`        | App-wide environment and endpoint constants           |
| `src/lib/`           | Shared backend logic and helpers                      |
| `src/types/`         | Central TS types for data models                      |

---

## 🎨 3. App Layer (React SPA)

| Folder                   | Purpose                                |
|--------------------------|----------------------------------------|
| `src/react-app/components/` | UI + feature components              |
| `src/react-app/hooks/`      | Shared React hooks (`useAuth`, etc.)|
| `src/react-app/pages/`      | Top-level routed pages              |
| `src/react-app/styles/`     | CSS tokens, themes, reset            |
| `src/react-app/contexts/`   | Context providers                    |
| `src/react-app/utils/`      | Utility functions for UI             |

---

## 🧪 4. Testing + Fixtures

| Folder             | Purpose                                  |
|--------------------|------------------------------------------|
| `tests/unit/`      | Component and function unit tests        |
| `tests/integration/` | Hono route handler tests               |
| `tests/e2e/`       | Full-stack UI tests (e.g. Playwright)    |
| `tests/_fixtures/` | Predefined test data (sessions, inventory) |

---

## 📄 5. Documentation + Operational State

| File/Folder                                  | Purpose                                              |
|----------------------------------------------|------------------------------------------------------|
| `README.md`                                  | Canonical project overview and onboarding guide      |
| `docs/architecture/SeshTracker_Architecture_and_Integration.md` | System architecture |
| `docs/testing/integration-kushobserver-verify.md`               | Auth API validation tests |
| `.internal/responses/response-to-kushobserver.md`               | Auth provider communication |
| `.cursor/RULES.md`                            | Enforced Cursor generation rules (this file)         |

---

## ❗ Enforcement Notes

- These files **must not be renamed, moved, or overwritten** without explicit team action
- Cursor must **never ignore** these paths when:
  - Generating new routes, types, hooks, or styles
  - Updating configuration or scripts
  - Running deployment or build tasks
- If any critical file is absent or malformed, Cursor should stop execution and prompt for user guidance

---

## ✅ Compliance Expectations

Cursor must:
- Inject all new logic into an approved folder from the lists above
- Log any changes to tracked config files
- Suggest updates to architectural docs if modifying integration, routes, or platform logic

---

_Last synced with `seshtrackerdev/sesh-tracker.com` on: {{INSERT_DATE_HERE}}_

---
> Source: [seshtrackerdev/sesh-trackeroldKUSH](https://github.com/seshtrackerdev/sesh-trackeroldKUSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
