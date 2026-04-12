## ragorchestraionstudio

> Rules for implementing the TVC-style admin backend as per the Admin Section Implementation Plan.


## Auth & session model

- Implement auth endpoints in `backend/src/routes/auth.ts` (or equivalent Express auth router):
  - `POST /auth/register` – create a user with email, password, first/last name; assign default role (e.g. ANALYST/PM); create session; return `{ user, accessToken, refreshToken }`.
  - `POST /auth/refresh` – accept a refresh token, verify, and issue a new access token.
  - `GET /auth/me` – return current user profile based on access token.
  - `PATCH /auth/profile` – update profile fields.
  - `POST /auth/profile/upload-image` – upload or store profile image path.
- JWT structure:
  - Access token: 15-minute TTL, payload `{ id, email, role, sessionId }`.
  - Refresh token: 7-day TTL, payload `{ id, sessionId }`.
  - Use a separate `JWT_REFRESH_SECRET` (or clearly separate config) from the access token secret.

## Prisma models

- Extend `User` with profile and org fields:
  - `firstName`, `lastName`, `displayName`, `profileImage`, `isActive`, `lastLoginAt`, `microsoftId`, `jobTitle`, `department`, `officeLocation`, `companyName`, `teamId`.
- Sessions:
  - Add a `token` (UUID) and `lastActivity` to the `Session` model.
  - On logout, mark `isActive = false`.
- Roles & teams:
  - If using a `Role` table, follow the spec:
    - `name`, `displayName`, `description`, `color`, `permissions` (JSON), `isSystemRole`, `userCount`.
  - Teams:
    - `Team` and `UserTeam` as per spec, with hierarchy and membership relations.

## Admin routes

- Add or extend `backend/src/routes/admin.ts` (or similar) with:
  - **Users**:
    - `GET /admin/users?search=&role=` – filtered list.
    - `GET /admin/users/:id` – detail.
    - `POST /admin/users`, `PUT /admin/users/:id`, `DELETE /admin/users/:id`.
  - **Roles**:
    - `GET/POST/PUT/DELETE /admin/roles`; block delete for system roles.
  - **Teams**:
    - `GET/POST/PUT/DELETE /admin/teams`.
    - `POST /admin/teams/:teamId/members` and `DELETE /admin/teams/:teamId/members/:userId`.
  - **Sessions**:
    - `GET /admin/sessions?includeInactive=`.
    - `GET /admin/sessions/:id/activity` (audit trail).
    - `DELETE /admin/sessions/user/:userId` – terminate all for user.
    - `POST /admin/sessions/cleanup` – cleanup expired sessions.

## Integrations, views, data integration, observability

- When implementing later phases:
  - **Integrations**: follow `Integration` and `IntegrationConfiguration` models, with endpoints to list/search/configure/test/disconnect integrations.
  - **Hierarchy views**: implement `HierarchyView` with CRUD and publish/default toggles.
  - **User preferences**: use `UserPreference` to centralize per-user defaults.
  - **Data integration**: `TableMapping`, `ImportFieldMapping`, `TransformationRule` per spec with dedicated routes.
  - **Observability**: add `ApiMetric`, `AiUsageMetric`, `AlertRule`, `AlertHistory`, `DbSnapshot` and their associated `/admin/observability/*` endpoints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KinshukON)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KinshukON)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
