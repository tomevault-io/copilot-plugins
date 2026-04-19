## openpolicy-merge-platform-v2

> - **BEFORE implementing ANY new functionality, ALWAYS check `/legacy` directory first.**

# =====================
# Cursor Global Rules for OpenPolicy Monorepo
# =====================

## 🚨 FUNDAMENTAL RULE #1: NEVER REINVENT THE WHEEL 🚨
- **BEFORE implementing ANY new functionality, ALWAYS check `/legacy` directory first.**
- If similar code exists in `/legacy`, copy it over and adapt/refactor it to meet current requirements.
- Do NOT write new implementations from scratch if legacy code can be reused/modified.
- This applies to: parsers, APIs, utilities, data loaders, transformers, validators, etc.
- When copying from legacy: preserve original comments noting source, then document adaptations made.
- **THIS IS THE MOST IMPORTANT RULE - NEVER SKIP THIS STEP.**

## 1. Style & Language
- Use TypeScript strict mode for all TS projects. Avoid "any"; model domain types explicitly.
- Require Python >=3.11; enforce Ruff + Black formatting; keep functions pure where possible.
- All new APIs and data loaders must have tests first (TDD-leaning).
- Use ES6+ features, arrow functions, and async/await in JavaScript projects.
- Prefer immutability and functional decomposition throughout.
- Strict type annotations or JSDoc as per language.
- Avoid duplicate code via abstraction; handle edge cases and input validation rigorously.

## 2. Architecture Principles
- API-first: openapi.yaml is updated before implementing endpoints.
- Do not break existing contracts; introduce new fields behind feature flags, avoid breaking changes.
- Ingestion must be idempotent; all loaders should support upsert semantics.

## 3. Safety & Security
- Never delete files in `/legacy`; instead, propose replacements under `/services` or `/apps` and mark deprecated code in `/docs/REFERENCE/omitted.md`.
- Never commit secrets or credentials; always use `.env` and GitHub Actions secrets.
- Sanitize all inputs (prevent injection/XSS).
- Use environment variables for API keys or secrets and do not log sensitive data.

## 4. Cursor Behavior & AI Instructions
- Treat `instructions.md` as the single source of truth for task scope.
- When uncertain, auto-generate a design note in `docs/ADR/` with pros/cons/decision.
- Provide clear plan or reasoning before code changes.
- Ask clarifying questions on ambiguous prompts.
- Always include unit tests alongside code changes.
- Suggest alternative patterns with rationale.
- Avoid fabrications; seek factual or confirmed knowledge.

## 5. Testing & Quality Thresholds
- Statement coverage must be 85%+ for `services/api-gateway` and `services/etl`.
- Branch coverage must be 95%+ for critical modules (parsers, normalizers).
- Generate integration and unit test cases for all major changes.
- Run all test suites post-merge; note and mark failed files.

## 6. Version Control & Commit Messages
- Follow Conventional Commits: `feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`, `ci:`.
- Write concise, meaningful messages; explain rationale for major changes in PRs or commit bodies.

## 7. Codebase Organization & Repo Merging
- Workspace consists of multiple merged repos; maintain traceability of migrated code in comments and commit messages.
- Standardize all folder/file naming; flatten redundant directories (merge multiple `/src`/`/test` etc.).
- Move shared logic/utilities to `common/` at root level.
- Eliminate duplicate code by suggesting unification or abstraction.
- Refactor and upgrade dependencies to a single, unified system; standardize config formats.
- Mark deprecated/legacy code with clear isolation (e.g., `/legacy`), and explicit deprecation notes.

## 8. AI-Assisted Refactoring & Conflict Handling
- Auto-detect and list conflicts: duplicate files/classes/functions/module names.
- Use path/glob rules to trigger language-appropriate refactors (e.g., JS rules in `/frontend`, Python in `/backend`).
- For every duplicated/conflicting fragment, propose the best candidate and document resolution process.
- Always merge documentation (README, LICENSE, CONTRIBUTING) and update links/source attribution as needed.
- Preserve git history where possible; otherwise, note source repo/path in headers.

## 9. Testing, CI, and Linting in Unified Repo
- Upgrade, merge, and unify test runners (Jest, Pytest, etc.), linters, and formatters for a single workflow.
- No merged code should break lint or test on CI.
- Run all test suites after merge; summarize conflicts and next steps.
- Propose test or linter migration where disparate tools exist.

## 10. Documentation & Traceability
- Every moved module/file should reference original source in comments or commit logs.
- Maintain a migration report in `docs/`: list moved/transformed files, structural changes, and outstanding/revisit items.
- Insert links to original repo URLs/commits for auditability where practical.

## 11. Collaboration & Communication
- When changes may impact other teams or systems, document the impact and tag relevant owners.
- Clearly document deprecation, API, or config/environment changes.
- Use professional, concise, and friendly communication; avoid jargon unless defined.

## 12. Advanced Merge & Workflow Practices
- Before code merging, produce or update architectural map/diagram of both the source and target repos.
- Break large merges into phases: 1) Move code, 2) Unify structure, 3) Resolve conflicts, 4) Integrate tooling, 5) Update docs.
- Use feature branches/worktrees for each merge phase for safety and clarity.
- Keep AI context up to date by opening relevant files and closing unrelated tabs.
- Regularly review and self-improve these rules based on results.

## 13. API & External Integration
- Handle API errors with retries, timeouts, and clear logs.
- Respect and document rate limits.
- Use explicit environment variable configs for API keys and secrets; do not hardcode.

## 14. UI/UX & Accessibility (If Relevant)
- Ensure responsive design and semantic HTML.
- Apply ARIA roles and keyboard navigation.
- Provide fallback UI for loading, error, empty states.
- Prioritize accessibility: color contrast, focus indicators, readable fonts.

## 15. Miscellaneous & Continuous Improvement
- Seek feedback for uncertain or major changes.
- Propose improvements and refactorings beyond immediate requests when justified.
- Avoid off-topic code generation or side conversations.
- After merge, schedule a post-mortem in docs on challenges and automation wins/gaps.
- Keep this `.cursorrules` file updated as the monorepo evolves.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ashish-tandon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
