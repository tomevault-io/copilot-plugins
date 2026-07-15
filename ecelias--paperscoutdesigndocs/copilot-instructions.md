## paperscoutdesigndocs

> This file guides AI agents that create or modify the PaperScout.ai codebase.

# CLAUDE.md

# Repository Instructions for AI Coding Agents

This file guides AI agents that create or modify the PaperScout.ai codebase.
PaperScout.ai is a web application that sends daily or weekly updates about
recent literature in a user's research field, summarizes selected articles with
AI, and teaches young researchers how search strategies work.

Before doing any work in this repository, read these files in full when they
exist:

1. `README.md`
2. `PLANNING.md`, `PAPERSCOUT.md`, or the project planning Markdown document
3. `LICENSE`
4. `CODING_STANDARDS.md`
5. `DEVELOPER.md`
6. `QUICKSTART.md`
7. `docs/architecture/` documents
8. `docs/api/` documents
9. `docs/user-guides/` documents

The project planning document is the product source of truth. It defines the
mission, license, functional requirements, non-functional requirements,
technology choices, planned units, testing expectations, and integration tests.

`AGENTS.md` is the execution source of truth for AI agents. It defines how an
agent should plan, scope, implement, test, and document changes.

When instructions conflict, use this order of authority:

1. Explicit user request for the current ticket.
2. This `AGENTS.md` file.
3. `CODING_STANDARDS.md`, if present.
4. The PaperScout.ai planning document.
5. `README.md` and `DEVELOPER.md`.
6. Existing code patterns.

Never remove the project license or copyright notices.

---

## Project Mission

PaperScout.ai helps young researchers stay current with recently published
literature. The system must:

1. Let users register, log in, and manage their profiles securely.
2. Let users create, edit, delete, and review literature search queries.
3. Retrieve relevant articles from PubMed and arXiv.
4. Select the most relevant articles based on search terms, preferences, and
   feedback.
5. Generate concise, readable AI summaries for selected articles.
6. Send scheduled updates by the user's selected delivery method.
7. Store search history and returned article metadata.
8. Ask for optional feedback on result relevance and summary usefulness.
9. Use feedback to improve future search strategies and summarization.
10. Show the search strategy used for each update so users can learn.

The product is for users who may be new to literature search. Favor clear
interfaces, obvious workflows, simple language, and helpful explanations over
feature density.

---

## License Requirements

PaperScout.ai is licensed under the GNU Affero General Public License v3.0 or
later.

All agents must:

1. Preserve AGPL license text and notices.
2. Avoid adding dependencies with licenses that conflict with AGPL-3.0-or-later.
3. Prefer open-source libraries and frameworks.
4. Document any dependency license concerns in the implementation plan.
5. Never replace the license without explicit approval.

---

## Core Technology Stack

Use the stack defined in the planning document unless an approved plan says
otherwise.

### Frontend

1. JavaScript.
2. React.
3. CSS.
4. Jest.
5. React Testing Library.
6. Cypress for browser-level end-to-end tests.

### Backend

1. Python.
2. FastAPI.
3. Pytest.
4. `unittest.mock` for external dependency mocking.
5. Pydantic models for request and response validation.

### Databases

1. SQLite for local development when needed.
2. PostgreSQL for production and integration testing.
3. Database constraints for uniqueness, nullability, and foreign keys.
4. Rollback or isolated database state for every database test.

The planning document mentions Django-style database tests in one section, but
FastAPI is the selected backend framework. Do not add Django unless an approved
plan explicitly changes the backend architecture.

### External Services

1. PubMed E-utilities for biomedical literature retrieval.
2. arXiv API for preprint retrieval.
3. AI summarization provider, configured through environment variables.
4. Postmark for email delivery, if approved and configured.
5. Plivo for SMS delivery, if approved and configured.
6. CronJob-style scheduled jobs for update delivery.

Do not hard-code external service credentials.

---

## Required Project Structure

Use the project structure from the planning document unless an approved plan
changes it. Keep implementation files and tests aligned by unit.

```text
project-root/
├── source/
│   ├── unit-01-landing-page/
│   ├── unit-02-about-app-page/
│   ├── unit-03-about-developer-page/
│   ├── unit-04-sign-up-page/
│   ├── unit-05-login-page/
│   ├── unit-06-user-dashboard/
│   ├── unit-07-user-profile/
│   ├── unit-08-search-history-page/
│   ├── unit-09-new-query-page/
│   ├── unit-10-previous-search-page/
│   ├── unit-21-search-history/
│   ├── unit-22-user-feedback/
│   ├── unit-23-update-notifications/
│   ├── unit-24-pubmed-api/
│   ├── unit-25-arxiv-api/
│   ├── unit-26-article-selection-summarization/
│   ├── unit-27-search-query-optimization/
│   ├── unit-28-update-formatting/
│   ├── unit-29-admin-capabilities/
│   ├── unit-30-access-scheduler-db/
│   ├── unit-31-access-user-db/
│   ├── unit-32-access-search-history-db/
│   ├── unit-33-access-feedback-db/
│   ├── unit-34-update-scheduler-db/
│   ├── unit-35-update-user-db/
│   ├── unit-36-update-search-history-db/
│   ├── unit-37-update-feedback-db/
│   ├── unit-38-encrypt-password/
│   ├── unit-39-create-scheduler-item/
│   ├── unit-40-create-user/
│   ├── unit-41-create-search-history-item/
│   └── unit-42-create-feedback-item/
├── test/
│   ├── unit_test/
│   └── integration_test/
├── docs/
│   ├── architecture/
│   ├── api/
│   └── user-guides/
├── scripts/
│   ├── build/
│   ├── deploy/
│   └── test/
├── config/
│   ├── development/
│   ├── staging/
│   └── production/
├── .gitignore
├── README.md
└── package.json
```

Agents may add conventional application subdirectories inside the unit folders
when useful, but must keep the unit mapping obvious.

---

## Unit Boundaries

A unit means different things depending on category.

### Frontend Units

A frontend unit is a full page or major page component. Examples:

1. Landing page.
2. About app page.
3. About developer page.
4. Sign-up page.
5. Login page.
6. User dashboard.
7. User profile.
8. Search history page.
9. New query page.
10. Previous search page.

Each frontend unit must include:

1. UI component code.
2. Routing behavior when applicable.
3. Form validation when applicable.
4. Loading and error states when data is fetched.
5. Accessibility behavior.
6. Unit tests.

### Backend Units

A backend unit is usually a class, service, route group, or workflow. Examples:

1. User login.
2. User sign-up.
3. Dashboard display.
4. New query handling.
5. Profile management.
6. Admin capabilities.

Each backend unit must include:

1. Clear request and response models.
2. Validation behavior.
3. Service-level domain logic.
4. Explicit error handling.
5. Tests for normal, edge, and failure cases.

### AI, Messaging, and Database Units

AI, messaging, and database units are granular functions or adapters because
these areas carry reliability, privacy, and security risk.

Examples:

1. PubMed request builder.
2. arXiv response parser.
3. Article relevance selector.
4. Summary generator.
5. Update formatter.
6. Password hasher.
7. Create user record.
8. Update scheduler record.
9. Create feedback record.
10. Send email notification.

Keep I/O adapters separate from pure domain transformations.

---

## Planned Unit Index

Implement units according to the planning document. The intended units are:

1. Landing Page.
2. About App Page.
3. About Developer Page.
4. Sign Up Page.
5. Login Page.
6. User Dashboard.
7. User Profile.
8. Search History Page.
9. New Query Page.
10. Previous Search Page.
11. User Database.
12. User Feedback Database.
13. Search History Database.
14. Scheduler Database.
15. Dashboard Interactivity.
16. Dashboard Display.
17. User Login.
18. User Sign Up.
19. New Query.
20. User Profile backend workflow.
21. Search History backend workflow.
22. User Feedback backend workflow.
23. Update Notifications.
24. PubMed API.
25. arXiv API.
26. Article Selection and Summarization.
27. Search Query Optimization.
28. Update Formatting.
29. Admin Capabilities.
30. Access Scheduler Database.
31. Access User Database.
32. Access Search History Database.
33. Access Feedback Database.
34. Update Scheduler Database.
35. Update User Database.
36. Update Search History Database.
37. Update Feedback Database.
38. Encrypt Password.
39. Create Scheduler Item.
40. Create User.
41. Create Search History Item.
42. Create Feedback Item.

If a requested feature spans multiple units, state that clearly in the plan and
implement only the approved slice.

---

## Engineering Principles

All code must be:

1. Composable.
2. Testable.
3. Small in scope.
4. Secure by default.
5. Explicit about validation and error handling.
6. Deterministic wherever practical.
7. Accessible in user-facing areas.
8. Clear enough for humans and AI agents to reason about.
9. Aligned with the documented unit boundaries.
10. Limited to the approved ticket scope.

Prefer pure functions for domain logic.

Use immutable data structures where practical.

Avoid hidden mutation, mutable global state, implicit side effects, and
unreviewed background work.

Isolate side effects behind clearly named boundary functions. Side effects
include:

1. Network access.
2. Database reads and writes.
3. Filesystem access.
4. Email and SMS delivery.
5. Scheduler execution.
6. AI model calls.
7. Time-dependent behavior.
8. Randomness.
9. Authentication token generation.

Pass time, randomness, and configuration into functions when deterministic
testing would otherwise be difficult.

Do not use `assert` for runtime argument validation. Use explicit exceptions
such as `ValueError`, `TypeError`, or precise domain exceptions.

Catch specific exceptions. Avoid bare `except:`. Avoid catching `Exception`
unless re-raising as a precise domain exception with useful context.

---

## Ticket-Based Workflow

Treat every requested feature, bug fix, refactor, documentation update, test
change, dependency update, or configuration change as a single JIRA-style
ticket.

Work on only one chunk of functionality at a time.

Before writing code, produce a numbered implementation plan for review.
Stop after producing the plan. Do not write code until the plan is approved.

The plan must include:

1. Task summary.
2. Units affected.
3. Functional requirements.
4. Non-functional requirements.
5. Acceptance criteria.
6. Files expected to change.
7. New or modified functions, modules, classes, components, routes, or types.
8. Database schema or migration changes.
9. External services touched.
10. Security and privacy considerations.
11. Test strategy.
12. Documentation updates.
13. Dependency/version research.
14. Docker or Docker Compose build/run plan.
15. Edge cases and assumptions.
16. Explicitly out-of-scope items.

If requirements are ambiguous, document the assumption in the plan.

---

## After Plan Approval

Once the plan is approved:

1. Implement only what the approved plan describes.
2. Do not introduce unrelated refactors.
3. Do not introduce unapproved dependencies.
4. Do not alter public behavior outside the approved scope.
5. Do not silently expand database schema.
6. Do not silently add external API calls.
7. Add or update tests for all behavior changes.
8. Update documentation only when required by the approved scope.
9. Preserve documented unit boundaries.
10. Preserve the approved scope exactly.

If implementation reveals that the approved plan is incomplete, stop and ask
for approval to revise the plan.

---

## Version Control Workflow

Use the branch model from the planning document.

1. `main` contains only production-ready code.
2. `develop` is the feature-integration branch.
3. Feature branches branch from `develop`.
4. Bug branches branch from `develop`.
5. Rebase feature and bug branches onto `develop` regularly.
6. Use pull requests for merges into `develop`.
7. Use pull requests for releases from `develop` into `main`.
8. Require passing automated tests before merge.

Use Semantic Versioning in `MAJOR.MINOR.PATCH` format:

1. Increment `MAJOR` for incompatible API changes.
2. Increment `MINOR` for backward-compatible functionality.
3. Increment `PATCH` for backward-compatible bug fixes.

---

## Python Style Requirements

Follow `CODING_STANDARDS.md` when present. Default expectations:

1. Use `from __future__ import annotations` where useful.
2. Use full package imports where practical.
3. Avoid wildcard imports.
4. Group imports in this order:
   1. Future imports.
   2. Standard library.
   3. Third-party packages.
   4. Local modules.
5. Use modern type annotations such as `str | None`, `list[int]`, and
   `dict[str, str]`.
6. Prefer `Sequence`, `Mapping`, and other abstract collection types for input
   parameters when appropriate.
7. Use `TypeAlias` for type aliases.
8. Use descriptive type variables such as `_ArticleT`.
9. Avoid mutable default arguments.
10. Keep functions small and focused.
11. Prefer refactoring functions that grow beyond roughly 40 lines.
12. Use one statement per line.
13. Use four-space indentation.
14. Limit lines to 88 characters unless a documented exception applies.
15. Use two blank lines between top-level definitions.
16. Use one blank line between methods.
17. Use f-strings or logging parameter interpolation appropriately.
18. For logging, call logging functions with a pattern string and parameters,
    not f-strings.
19. Use context managers for files, sockets, database sessions, and other
    stateful resources.
20. Place executable script logic in `main()`.
21. Guard executable scripts with:

```python
if __name__ == "__main__":
    raise SystemExit(main())
```

---

## FastAPI Requirements

Backend APIs must be designed around clear contracts.

1. Use Pydantic models for request bodies, query parameters where useful, and
   response payloads.
2. Validate user input at API boundaries.
3. Keep route handlers thin.
4. Put business logic in services.
5. Put database access in repositories or clearly named data-access modules.
6. Use dependency injection for database sessions, authenticated users,
   settings, and external clients.
7. Return appropriate HTTP status codes.
8. Avoid leaking internal exception details to users.
9. Log operational errors without exposing secrets or private user data.
10. Document endpoints in `docs/api/` when behavior changes.

---

## React Requirements

Frontend code must be simple, accessible, and easy to test.

1. Use React components with clear prop names.
2. Use controlled inputs for forms.
3. Validate forms before submitting.
4. Display useful loading, empty, success, and error states.
5. Keep data fetching separate from pure presentational components when
   practical.
6. Use React Testing Library to test user-observable behavior.
7. Use semantic HTML before custom widgets.
8. Add accessible labels to inputs and buttons.
9. Make keyboard navigation work for interactive elements.
10. Avoid exposing implementation details in tests.

Naming conventions:

1. React components use `PascalCase`.
2. Hooks use `useCamelCase`.
3. Utility functions use `camelCase` in JavaScript.
4. Routes should be clear and stable, such as `/login`, `/signup`,
   `/dashboard`, `/profile`, `/queries/new`, and `/search-history`.

---

## Functional-Programming Requirements

Prefer functional-programming style for domain logic.

Pure functions must:

1. Return the same result for the same inputs.
2. Avoid mutating caller-owned arguments.
3. Avoid reading or writing hidden global state.
4. Avoid I/O.
5. Avoid network calls.
6. Avoid filesystem calls.
7. Avoid database access.
8. Avoid time-dependent behavior unless time is passed in explicitly.
9. Avoid random behavior unless randomness is passed in explicitly.
10. Return new values instead of mutating existing values.

When side effects are required, isolate them in boundary functions and test them
with mocks or controlled fixtures.

Prefer dispatch tables over long `if`/`elif` chains when mapping keys to
operations.

Avoid clever, overly dense, or hard-to-debug functional code.

---

## Docstrings With Acceptance Criteria

For pure functions, service functions, validation functions, adapters, and
public APIs, include docstrings with explicit acceptance criteria.

Use this structure where applicable:

```python
def normalize_search_terms(raw_terms: list[str]) -> list[str]:
    """Return normalized search terms for literature retrieval.

    Acceptance criteria:
        1. Determinism: Same input returns the same ordered output.
        2. No mutation: Do not mutate caller-owned lists.
        3. Validation: Empty normalized terms raise `ValueError`.
        4. Normalization: Terms are stripped and empty entries are removed.
        5. Deduplication: Duplicate terms are removed case-insensitively.

    Args:
        raw_terms: User-provided search terms.

    Returns:
        Normalized search terms.

    Raises:
        ValueError: If no usable terms remain after normalization.
    """
```

Acceptance criteria must be specific, testable, and aligned with the approved
plan.

---

## Database Requirements

Database code must protect user data and preserve relationships between users,
queries, schedules, results, and feedback.

Required logical tables or equivalent models include:

1. Users.
2. User profiles or preferences.
3. Search history.
4. Scheduler entries.
5. Article result records or summary references.
6. Feedback.
7. Notification delivery logs.
8. Admin audit logs when admin actions are implemented.

Database work must:

1. Use migrations when schema changes are introduced.
2. Enforce unique user emails.
3. Store password hashes only; never store plaintext passwords.
4. Enforce foreign keys between users, searches, schedules, feedback, and
   notifications.
5. Validate required fields.
6. Use transactions for multi-step writes.
7. Roll back test data after each test.
8. Mask sensitive fields when returning user records.
9. Avoid logging private user data.
10. Include tests for insert, update, retrieve, delete, and constraint behavior.

When a query is deleted, remove or disable future scheduler entries as required
by the product workflow, but preserve historical records unless an approved plan
requires deletion.

---

## Authentication and Security Requirements

Security is a core non-functional requirement.

1. Hash passwords with an approved modern password hashing library.
2. Never store plaintext passwords.
3. Never return password hashes to the frontend.
4. Use secure token/session handling.
5. Store secrets only in environment variables or approved secret stores.
6. Encrypt sensitive data at rest when supported by the deployment environment.
7. Use TLS in deployed environments.
8. Validate and sanitize all user input.
9. Protect routes that require authentication.
10. Restrict admin actions to authorized admin users.
11. Avoid logging emails, phone numbers, tokens, search details, or feedback
    content unless the approved plan explicitly defines safe logging.
12. Treat user research interests as sensitive personal data.

Authentication tests must cover:

1. Successful sign-up.
2. Duplicate email rejection.
3. Invalid input rejection.
4. Successful login.
5. Failed login.
6. Logout or token invalidation behavior.
7. Protected route access denial.
8. Password hash verification.

---

## Literature API Requirements

PubMed and arXiv integrations must be reliable and testable.

1. Keep request construction separate from network execution.
2. Keep response parsing separate from network execution.
3. Use timeouts for external requests.
4. Handle network errors, API errors, malformed responses, and empty results.
5. Normalize article metadata into an internal article type.
6. Preserve source database information.
7. Preserve article links.
8. Preserve DOI when available.
9. Preserve title, authors, abstract, publication date, and source metadata
   when available.
10. Mock external APIs in unit and integration tests.

Do not run live PubMed or arXiv requests in unit tests.

---

## Search Strategy Transparency Requirements

Every user-facing update must be able to show the search strategy used.

Store or derive enough information to explain:

1. Original user-provided terms.
2. AI-suggested or optimized terms, when used.
3. Whether AI assistance was enabled, partially disabled, or disabled.
4. Databases searched.
5. Query strings sent to each external database.
6. Filters used, such as date range, article count, or source.
7. Retrieval timestamp.
8. Selection criteria for delivered articles.
9. Any feedback signals used to refine the search.

Do not hide AI query expansion from users.

---

## AI Integration Requirements

AI features must support the educational goals of PaperScout.ai.

1. Users must be able to fully or partially opt out of AI integration.
2. AI-generated query suggestions must be clearly labeled.
3. AI-generated summaries must be clearly labeled.
4. Summaries must be concise, readable, and appropriate for young researchers.
5. Summaries must not fabricate article claims.
6. If metadata or article content is missing, state that it is unavailable or
   use a safe fallback.
7. Keep prompt construction separate from model invocation.
8. Keep model invocation behind a boundary function or client adapter.
9. Use structured summary outputs when practical.
10. Validate summary length and required fields.
11. Include article links and metadata in formatted updates.
12. Never send unnecessary personal data to the AI provider.

AI tests must cover:

1. Article relevance selection.
2. Summary structure.
3. Summary length.
4. Missing abstract fallback.
5. Malformed model output fallback.
6. AI opt-out behavior.
7. Feedback-informed search or selection behavior when implemented.

---

## Update Formatting and Notification Requirements

Updates must be useful, readable, and safe to deliver.

1. Format updates for the selected delivery method.
2. Include article title, authors when available, source database, DOI when
   available, publication date when available, summary, and full article link.
3. Include a feedback prompt.
4. Include the option to view the search strategy.
5. Escape HTML and user-provided content.
6. Avoid confusing formatting for SMS.
7. Log delivery status without storing secrets.
8. Retry only when retries are explicitly designed and safe.
9. Ensure duplicate updates are not sent for the same scheduled run.

The system should process and deliver scheduled updates within two minutes of
the scheduled time in normal operating conditions.

---

## Scheduler Requirements

Scheduler behavior must be deterministic and auditable.

1. Store user frequency preferences.
2. Store `next_update` and `last_run` or equivalent fields.
3. Compute next run time in a testable function.
4. Use timezone-aware datetimes.
5. Make scheduled update execution idempotent.
6. Prevent duplicate scheduler entries for the same active query unless the
   approved schema allows them.
7. Update scheduler entries when queries are edited.
8. Remove or disable scheduler entries when queries are deleted.
9. Create search history entries when scheduled updates run.
10. Prompt for feedback after update delivery.

Scheduler tests must cover:

1. Daily schedules.
2. Weekly schedules.
3. Invalid frequency rejection.
4. Missing scheduler record behavior.
5. Query edit behavior.
6. Query delete behavior.
7. Concurrent scheduler update behavior.
8. Delivery log behavior.

---

## Admin Requirements

Admin capabilities must be protected and audited.

1. Admin-only routes must reject non-admin users.
2. Admin user management actions must be logged.
3. Feedback review actions must be logged.
4. Schedule override actions must be logged.
5. Destructive admin actions require explicit approved scope and tests.
6. Admin views must mask sensitive data unless there is an approved need.

Do not build broad admin features while implementing ordinary user workflows
unless the approved plan includes Unit 29.

---

## Accessibility and UX Requirements

PaperScout.ai is intended for young researchers who may be unfamiliar with
literature databases.

1. Use simple, direct page copy.
2. Explain technical search terms where useful.
3. Make primary actions obvious.
4. Keep forms short and readable.
5. Use accessible labels and headings.
6. Preserve keyboard navigation.
7. Provide clear success, empty, and error messages.
8. Avoid jargon in user-facing messages unless it is explained.
9. Make search history and past updates easy to find.
10. Make the search strategy explanation easy to understand.

Prefer WCAG 2.1 AA-compatible patterns where practical.

---

## Testing Requirements

Add or update tests for every behavior change.

### Unit Tests

Unit tests must verify:

1. Determinism.
2. No mutation of caller-owned inputs where applicable.
3. Correct return values.
4. Validation behavior.
5. Raised exceptions.
6. Edge cases listed in the approved plan.
7. Boundary behavior for I/O wrappers with mocks.
8. Database constraints and rollback behavior.
9. AI and external API fallback behavior.
10. Frontend rendering, routing, forms, responsiveness, and interactions.

Use:

1. Jest and React Testing Library for React components.
2. Pytest for Python logic, services, repositories, and API routes.
3. `unittest.mock` for external dependencies.

### Integration Tests

Integration tests must verify component interactions.

Required flows include:

1. User sign-up and login.
2. Profile management and scheduler update.
3. New search query submission and search history recording.
4. Automated literature retrieval and update delivery.
5. Search strategy transparency.
6. User feedback submission and feedback-based refinement.
7. Query edit and delete behavior.

Use controlled mock responses for PubMed, arXiv, AI, Postmark, and Plivo unless
an approved plan specifically calls for live-service testing.

### End-to-End Tests

Use Cypress for browser-to-API user workflows.

E2E tests should cover:

1. Sign up.
2. Log in.
3. Submit a new query.
4. View retrieved articles and summaries.
5. View search strategy.
6. Submit feedback.
7. Edit profile preferences.
8. Log out.

### Coverage

Maintain greater than 90% code coverage where practical. If a ticket cannot
meet that target, explain why in the plan and final notes.

Do not skip tests unless explicitly approved in the plan.

---

## CI/CD Requirements

Use GitHub Actions for automated testing and quality checks.

CI should run:

1. Python linting and formatting checks.
2. Python unit tests.
3. Backend API tests.
4. Frontend linting and formatting checks.
5. React unit tests.
6. Integration tests.
7. Cypress tests where practical.
8. Coverage reporting with Coveralls or the approved coverage service.

Tests must run automatically on pull requests.

---

## Dependency and Version Requirements

Before proposing, adding, removing, or changing dependencies:

1. Research the latest stable version.
2. Check license compatibility with AGPL-3.0-or-later.
3. Explain why the dependency is needed.
4. Document the selected version.
5. Include the change in the implementation plan.
6. Wait for approval before changing dependency files.

Do not add dependencies for convenience when standard-library or existing
project tools are sufficient.

---

## Docker Requirements

Use Docker or Docker Compose as the primary build and run workflow once the
codebase is bootstrapped.

Do not rely on local host execution as the only documented workflow.

Default expectations:

1. Backend Docker image uses an approved Python base image.
2. Frontend Docker image uses an approved Node base image.
3. PostgreSQL runs as a Docker Compose service for integration testing.
4. Environment variables are passed through Compose, not hard-coded.
5. Test commands can run inside containers.

A plan involving build or runtime changes must include:

1. Docker build command.
2. Docker Compose run command.
3. Backend test command.
4. Frontend test command.
5. Integration test command.
6. Required environment variables.

---

## Configuration Requirements

Keep configuration environment-specific.

Expected config areas:

1. `config/development/`
2. `config/staging/`
3. `config/production/`
4. `.env.example`
5. Docker Compose environment variables.

Expected environment variables may include:

1. `APP_ENV`
2. `DATABASE_URL`
3. `SECRET_KEY`
4. `OPENAI_API_KEY` or configured AI provider key
5. `NCBI_API_KEY`, if used for PubMed
6. `POSTMARK_API_TOKEN`
7. `PLIVO_AUTH_ID`
8. `PLIVO_AUTH_TOKEN`
9. `PLIVO_FROM_NUMBER`
10. `DEFAULT_UPDATE_TIMEZONE`

Never commit real secrets.

---

## Documentation Requirements

Maintain the documentation set defined in the planning document.

### `README.md`

`README.md` is the user-facing and quick-start guide. It should explain:

1. What PaperScout.ai does.
2. How to install or build it.
3. How to configure it.
4. How to run it.
5. How to use it.
6. Common examples.
7. Troubleshooting notes.
8. License information.

### `DEVELOPER.md`

`DEVELOPER.md` is the developer onboarding guide. It should include:

1. Project structure overview.
2. File-by-file or module-by-module explanation.
3. Table of available commands.
4. Command flags and parameters.
5. Expected command behavior.
6. Docker and Docker Compose workflows.
7. Testing instructions.
8. Development workflow notes.
9. External service mocking instructions.
10. Database migration instructions.

### `docs/architecture/`

Architecture documentation should include:

1. System overview.
2. Mermaid or PlantUML diagrams when useful.
3. Data flow diagrams.
4. Scheduler flow.
5. AI pipeline flow.
6. Security model.
7. Database model.

### `docs/api/`

API documentation should include:

1. Endpoint list.
2. Request and response schemas.
3. Authentication requirements.
4. Error responses.
5. Example payloads.

### `docs/user-guides/`

User guides should explain:

1. Creating an account.
2. Creating a search query.
3. Reading an update.
4. Understanding search strategies.
5. Providing feedback.
6. Managing profile and delivery preferences.

---

## When to Update Documentation

Update documentation whenever the approved change affects:

1. User behavior.
2. Developer workflow.
3. Commands.
4. Flags.
5. Parameters.
6. Setup steps.
7. Docker usage.
8. Docker Compose usage.
9. Dependency versions.
10. Project structure.
11. Database schema.
12. API endpoints.
13. Scheduler behavior.
14. AI behavior.
15. Search strategy transparency.
16. Notification behavior.
17. Security behavior.

Do not update unrelated documentation.

---

## Definition of Done

A ticket is done only when:

1. The approved scope is implemented.
2. Unit boundaries remain clear.
3. Tests are added or updated.
4. Tests pass in the documented workflow.
5. New behavior is documented when required.
6. External services are mocked in tests.
7. Security and privacy risks are addressed.
8. Accessibility requirements are addressed for user-facing changes.
9. Error handling and fallback behavior are tested.
10. No unrelated refactors are included.
11. No unapproved dependencies are introduced.
12. No secrets are committed.
13. The final response summarizes changed files, tests run, and any known
    limitations.

---

## Scope Control

Do not:

1. Write code before plan approval.
2. Implement functionality outside the approved plan.
3. Refactor unrelated code.
4. Add unapproved dependencies.
5. Change public behavior outside the approved scope.
6. Introduce hidden global state.
7. Add mutable module-level state unless explicitly approved.
8. Store plaintext passwords.
9. Log secrets or sensitive user data.
10. Skip tests for behavior changes.
11. Call live external services in unit tests.
12. Replace FastAPI with another backend framework without approval.
13. Replace React with another frontend framework without approval.
14. Change the license without approval.
15. Hide AI-generated query expansion or summary behavior from users.

---

## Role Expectation

Act like a senior software engineer building a secure educational research
product.

Be precise, conservative in scope, and quality-focused.

Favor clear, maintainable, well-tested code over clever implementations.

When uncertain, document the assumption in the plan before implementing.

Never expand scope silently.

---
> Source: [ecelias/PaperScoutDesignDocs](https://github.com/ecelias/PaperScoutDesignDocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
