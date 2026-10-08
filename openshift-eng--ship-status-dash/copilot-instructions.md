## security

> Authentication, authorization, and credential handling rules for Ship Status Dashboard


### Authentication model

The dashboard uses a dual-ingress architecture:

* **Public route** (`ship-status.ci.openshift.org`, port 8080) -- read-only API, no authentication required.
* **Protected route** (`protected.ship-status.ci.openshift.org`, port 8443) -- routes through an oauth-proxy that authenticates callers via Kubernetes `TokenReview`, sets `X-Forwarded-User`, and signs requests with an HMAC `GAP-Signature` header before proxying to the dashboard on loopback.

The oauth-proxy is the bearer-token authentication boundary. The dashboard (`cmd/dashboard/auth.go`) validates the `X-Forwarded-User` header and `GAP-Signature` HMAC to confirm the request passed through oauth-proxy untampered, then enforces authorization against `Owner.User`, `Owner.ServiceAccount`, and `Owner.RoverGroup`. Outage writes match those fields on the component. Team SLO workspace writes match them on that team's `team_slos` entry (`IsUserAuthorizedForTeamSLO`). Component owners do not grant team SLO write access. The dashboard never sees or validates bearer tokens directly. Trusted service accounts (configured in `trusted_delegators`) can act on behalf of a user by providing the `X-Acting-For` HTTP header; the auth middleware resolves the delegated identity before handlers run, so authorization and auditing use the delegated user transparently.

### Credential placement

Never mount secret tokens (service account tokens, API keys) on containers that accept unauthenticated inbound traffic. If a container is publicly accessible, it must not have access to credentials that grant write access to other services.

`jira_monitor` searches Jira anonymously. It only sees issues the site grants **Browse** to Anyone (TRT and OCPBUGS on `redhat.atlassian.net` do). Do not mount a Jira token on any Ship Status pod.

Services behind oauth-proxy (not publicly accessible) may hold credentials needed for downstream authenticated calls. This is acceptable because oauth-proxy ensures only authenticated callers can reach the service. Services that accept unauthenticated traffic must remain stateless and credential-free; the caller supplies their own bearer token, which is forwarded unmodified to oauth-proxy for authentication.

### Write endpoint authorization

All mutating API endpoints (create, update, delete) must be served exclusively on the protected route. The public route must never expose write operations, even behind application-level checks. Defense in depth: the oauth-proxy layer authenticates, the HMAC layer verifies request integrity, and the dashboard authorizes against the owner list for that resource. Outage mutations use the component `owners`. Team SLO item and link mutations use `team_slos[].owners` via `IsUserAuthorizedForTeamSLO`. Do not authorize a team SLO write from component owners when the caller is absent from that team's `team_slos` owners.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
