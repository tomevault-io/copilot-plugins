## feature-request

> Manage feature request intake, normalization, triage, queue operations, and API-only lifecycle workflows.


# Skills (Agent Task Catalog)

This file is the single skill registry for the repository. Each section is an actionable skill with a deterministic flow.

## Skill: `agent-onboard`
### Trigger
User asks for agent-facing setup or onboarding changes.

### Required Inputs
- Target feature area (`backend`, `frontend`, `tooling`, or `manifest`).
- Scope of changes and acceptance criteria.

### Command Flow
1. Read `./AGENTS.md`.
2. Read `./agents.json` if present.
3. Update only the requested sections.
4. Run relevant validation commands after edit.

### Expected Output
- Summary of updated sections.
- List of files changed.
- Validation command results.

### Failure Handling
- If required references are missing, stop and request the missing inputs before writing.
- If validation fails, include failing command output and propose next corrective step.

## Skill: `feature-request-operator` 
### Trigger
User asks an agent to read or mutate feature requests for a public board.

### Purpose
Operate on board data via API:
- read projects
- read requests/issues
- create requests
- add comments
- edit comments
- toggle upvotes

### Prompt Presets
- `portfolio-triage`: read-only queue snapshot across all projects owned by the authenticated user.
- `project-triage`: read-only queue snapshot for one `owner_handle/project_slug`.
- `project-implementation`: use one project's FeatureRequest issues as the source of truth for local repo implementation work.

### Required Inputs
- Base URL:
  - https://featurerequest.io
- Auth mode:
  - `Bearer <token>` flow.
  - If a raw API token is already provided in the prompt, use it directly in the `Authorization` header.
  - Do not call agent-token connect/refresh endpoints when a raw token is already available.
  - For write operations, token must be write-enabled (`can_write=true`).
- `owner_handle` (required for read/write paths under that board).
- Optional: `project_slug`.
- Input payloads:
  - request: `issue_type`, `title`, `description`, `priority`.
  - comment: `body`.
  - optional filters: `issue_type`, `status`, `priority`, `limit`.

### Required API Routes
- Web-session agent onboarding only:
  - `POST /api/auth/agent-token/connect`
  - `POST /api/auth/agent-token/refresh`
- Read projects:
  - `GET /api/projects`
  - `GET /api/owners/{owner_handle}/projects`
- Read issues:
  - `GET /api/owners/{owner_handle}/issues`
  - `GET /api/projects/{owner_handle}/{project_slug}/issues`
  - `GET /api/issues/{issue_id}`
- Create issue:
  - `POST /api/projects/{owner_handle}/{project_slug}/issues`
- Add comment:
  - `POST /api/issues/{issue_id}/comments`
- Edit comment:
  - `PATCH /api/issues/{issue_id}/comments/{comment_id}`
- Toggle upvote:
  - `POST /api/issues/{issue_id}/upvote/toggle`

### Command Flow
1. Validate auth credentials and action.
2. If a raw API token is present, use it directly as `Authorization: Bearer <token>`.
3. Use `POST /api/auth/agent-token/connect` or `/refresh` only for browser/session onboarding flows where no raw token has been provided.
4. If read requested:
   - for portfolio triage, call `GET /api/projects`, then issue lists for those projects.
   - for project-scoped triage or implementation, call `GET /api/projects/{owner_handle}/{project_slug}/issues` directly with optional filters.
   - if one issue is target, call issue detail.
5. If creating request:
   - call create issue endpoint with required body.
6. If adding comment:
   - call create comment endpoint with `{"body": "<text>"}`.
7. If editing comment:
   - call update comment endpoint with `{"body": "<text>"}`.
8. If upvote requested:
   - call upvote toggle endpoint and read returned `upvoted` + `upvotes_count`.
9. Return a normalized result object (see output format).
10. On failures, return error object with action and actionable recovery step.

### Project-Scoped Implementation Guidance
- Treat FeatureRequest as the ticket source of truth.
- Perform code changes only in the local repository/workspace where the coding agent is already running.
- Only read issues for the instructed `owner_handle/project_slug`; do not read or modify other projects.
- Pick at most one ready issue per run unless the user explicitly asks for more.
- Before code changes, produce a short implementation plan.
- Run relevant tests after edits.
- After implementation, add a concise comment back to the issue when write access is available.
- Do not mark an issue `done` automatically; reserve `done`/`closed` for merge or release confirmation.

### Expected Output Format
Return compact JSON:
```json
{
  "ok": true,
  "action": "read_issues|create_issue|add_comment|toggle_upvote",
  "request": {
    "method": "POST",
    "path": "/api/projects/{owner_handle}/{project_slug}/issues",
    "status": 201
  },
  "resource": {
    "owner_handle": "owner",
    "project_slug": "project",
    "issue_id": 123
  },
  "data": {},
  "meta": {
    "items_returned": 0,
    "created_at": "2026-02-27T12:00:00Z"
  }
}
```

For failures:
```json
{
  "ok": false,
  "action": "add_comment",
  "request": {...},
  "error": {
    "status_code": 400,
    "message": "Message rejected by moderation"
  },
  "next_step": "Retry with cleaner content or ask user for a revised message."
}
```

### Failure Handling
- `401`: refresh or request new credentials and retry.
- `403`:
  - for read-only token on write action, instruct token replacement with `can_write=true` before retry.
- `400`: log payload validation/moderation failure and stop; do not retry silently.
- `404`: validate `owner_handle`, `project_slug`, or `issue_id` input.
- `503`: retry with backoff for transient moderation/provider failures.
- Never auto-delete or mutate any resource unless `action` explicitly asked.

### Prompt Examples
- Read-only daily triage: read authenticated user's projects, summarize active requests, and return `Queue Snapshot`, `Priority Decisions`, `Active Follow-ups`, `Risks and Blockers`, and `Next Checkpoint`.
- Project-specific planning: read only `owner_handle/project_slug`, identify the next ready issue, and produce an implementation plan without editing code.
- Project-specific implementation with tests: read only `owner_handle/project_slug`, pick at most one ready issue, implement in the current local repo, run relevant tests, and comment back with results when write access is available.
- Release follow-up: after merge or release confirmation, add a closure comment and update the issue status only when explicitly asked.

## Skill: `user-request-manager`
### Trigger
User asks an agent to manage incoming requests as a queue (visibility, prioritization, ownership, follow-up, closure).

### Required Inputs
- API access token and scope.
- Request source input (list, ticket export, or pasted requests).
- Optional SLA or deadline policy.

### Command Flow
1. Ingest requests from provided source.
2. Normalize each item into an API-backed record:
   - `issue_id`, `owner_handle`, `project_slug`, `issue_type`, `title`, `description`, `priority`, `status`.
3. Deduplicate overlapping requests and keep one canonical item.
4. Triage by urgency/impact and assign `P0`-`P3`.
5. Maintain one prioritized queue sorted by priority and due date.
6. For each active request, include explicit next action in report output (do not persist non-model fields as API attributes).
7. Produce concise checkpoint updates (new/changed/blocked/due soon/next actions).
8. Close request only after expected outcome is met, with closure note.

### Expected Output
- `Queue Snapshot` table:
  - `issue_id | summary | priority | status | owner_handle/project_slug | next action`
- `Priority Decisions`
- `Active Follow-ups`
- `Risks and Blockers`
- `Next Checkpoint`

### Failure Handling
- Missing permissions/scope: return blocking auth note and exact missing scope.
- Missing required request fields: mark item `waiting`, request only missing fields.
- API rate-limit/transient failure: retry with bounded backoff and report partial progress.
- Never perform code or config edits from this skill; operate only through exposed request APIs.

---
> Source: [onurmatik/feature-request](https://github.com/onurmatik/feature-request) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
