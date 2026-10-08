## backend

> Go backend coding standards for Ship Status Dashboard


* Follow idiomatic Go practices.
* After making changes, always run `gofmt -w` on modified files to ensure proper formatting.
* Use GORM conventions for database models and queries.
* Authentication uses HMAC signature verification -- never bypass `SKIP_AUTH` in production paths.
* Outage modifications must go through the audit logging system (`outage_audit_logs` table).
* All mutating endpoints (create, update, delete) must be served exclusively on the protected route. The oauth-proxy is the bearer-token authentication boundary; the dashboard is the sole application-level enforcement point for HMAC validation and authorization on all write operations.
* The SPA handler injects Open Graph metadata into `index.html` for link previews (Slack and other clients that read OG tags). Route patterns in `metaRoutes` (`cmd/dashboard/meta.go`) mirror the frontend's React Router definitions. When adding a new frontend route, add a corresponding `metaRoutes` pattern so link previews render correctly.
* Team SLO workspace rows are keyed by `(kind, schema_version)`. JSON schemas live in `pkg/slo/schema`. Reject unknown versions. Ship a new schema file and renderer for a new version. Do not change a published schema in place.
* A team has at most one workspace (`DashboardConfig.ValidateTeamSLOs`).
* Team SLO outage wells use `team_slos[].slo_components` (component slugs). Each slug must be a component with `slo_component: true`, and each such component must be listed once. Do not join those wells on `ship_team`.
* `cmd/seed-slo` loads sample workspace rows for local dev and e2e. It is not a production writer.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
