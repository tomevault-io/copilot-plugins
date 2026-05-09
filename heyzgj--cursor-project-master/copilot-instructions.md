## 200-quality

> The Quality Guardian. It enforces a non-negotiable set of engineering commandments including Contract-First, TDD, code cleanliness, and security baselines. It acts as the project's automated QA lead and code reviewer.

# 200 · Quality Guardian: The Engineering Commandments

## CONTEXT
These commandments are the absolute law governing all code generation. They are invoked during the `EXECUTE & VERIFY` step of the `100-engine.mdc` task loop. Failure to comply with any of these commandments constitutes a failed task attempt and **MUST** trigger a reflection cycle.

---

### COMMANDMENT I: THE CONTRACT IS LAW (Contract-First)

1.  **Single Source of Truth**: The Zod schemas defined in `./schemas/` are the **immutable and sole source of truth** for all data entities and API payloads.
2.  **No Manual Implementations**: You are **STRICTLY PROHIBITED** from manually writing TypeScript types, database models (e.g., Prisma schema), or API validation logic for any entity defined in the schemas.
3.  **Mandatory Code Generation**: You **MUST** use the designated script (e.g., `npm run generate:types`) to transform Zod schemas into all necessary code artifacts. If a schema is modified, this script **MUST** be re-run before any other code is written.

### COMMANDMENT II: THOU SHALT WRITE TESTS FIRST (TDD)

1.  **The Unskippable Cycle**: For any task of `type: feature` or `type: bug`, the **Red-Green-Refactor** cycle is mandatory.
    1.  **RED**: Before writing any implementation, create the minimum number of unit or E2E tests that will fail because the feature is missing.
    2.  **GREEN**: Write the absolute minimum amount of code necessary to make the failing tests pass.
    3.  **REFACTOR**: Clean up the code without changing its functionality.
2.  **Acceptance Criteria as Tests**: Every item in a task's `Acceptance Criteria` checklist **MUST** be covered by at least one verifiable test case.
3.  **Visual & Design Compliance**: For any task that touches the UI (`ui_review: true`), E2E tests (using Playwright) **MUST** also validate compliance with the visual specifications defined in `/docs/DESIGN_SPEC.md`, including layout, component usage, and style guide adherence.

### COMMANDMENT III: THOU SHALT NOT HARDCODE

1.  **Sanctity of `/config`**: All environment-specific variables (URLs, API Keys, Ports) and magic constants **MUST** reside exclusively in the `/config` directory and be loaded from there.
2.  **Zero Tolerance**: A `grep` for common hardcoded patterns (e.g., `localhost`, `http://`, API key prefixes) is a mandatory part of the pre-commit check. Any findings constitute a failure.

### COMMANDMENT IV: CLEANLINESS IS NEXT TO GODLINESS (Linting & Formatting)

1.  **Mandatory Checks**: Before any `git commit`, the commands `eslint --max-warnings=0` and `prettier --check .` **MUST** be executed and pass.
2.  **Zero Warnings, Zero Errors**: A single warning or error from either tool constitutes a failure. You must fix all reported issues before proceeding.

### COMMANDMENT V: THOU SHALT HAVE NO CRITICAL VULNERABILITIES (Security)

1.  **Mandatory Audit**: Before any `git commit` that modifies `package.json` or other dependency files, the command `npm audit --production` (or equivalent) **MUST** be executed.
2.  **Zero Tolerance Gate**: The presence of any **High** or **Critical** level vulnerabilities constitutes a failure. You must find a safe version, an alternative package, or document the risk in a `/docs/DECISION_LOG.md` entry if no alternative exists.

### COMMANDMENT VI: THOU SHALT MAINTAIN COVERAGE (Continuous Testing)

1.  **The 70% Threshold**: The overall test line coverage, as measured by the CI pipeline, **MUST NOT** fall below 70%.
2.  **Automatic Test Regeneration Trigger**: A new `regen-tests` task **MUST** be automatically created and prioritized if:
    *   The coverage drops below the 70% threshold after a task is completed.
    *   Any file within `./schemas/` is modified (as this changes the data contract).
    *   An entire Epic is completed (as a final integration and regression check).

---
> Source: [heyzgj/cursor-project-master](https://github.com/heyzgj/cursor-project-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
