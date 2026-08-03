## kube-argus

> Guidance for future Claude sessions on this repo. Read this first before making changes.

# kube-argus — Claude working notes

Guidance for future Claude sessions on this repo. Read this first before making changes.

## What this is

Real-time Kubernetes dashboard shipped as a single Go binary + Helm chart. In-cluster deployment, OIDC / Google SSO auth, JIT exec workflow, Slack + generic webhook notifications, cost/spot advisor, AI-diagnose via LLM gateway, audit trail persisted to ConfigMaps. Maintained by @manishchaudhary101. Public repo: `github.com/manishchaudhary101/kube-argus`. ArtifactHub: `artifacthub.io/packages/helm/kube-argus/kube-argus`. Landing page: `manishchaudhary101.github.io/kube-argus`.

## Architecture (post-v1.2.8 reorganisation)

```
cmd/server/main.go              — 238-line wire-up ONLY (env → clients → DI → mux → ListenAndServe)

internal/
├── httpx/                      — response helpers (JSON, Error, K8sError, JSONGz, WsUpgrader, WsCheckOrigin)
├── auth/                       — sessions, OIDC, Google SSO, RequireAdmin, IsAdmin, ClientIP, middleware
├── audit/                      — trail, dedup, retention, ConfigMap persistence, WebSocket presence broadcast
├── jit/                        — JIT store, approve/deny/revoke/expiry, ConfigMap persistence
├── notify/                     — Slack + generic webhook, HMAC signing, event filtering
└── api/                        — every HTTP handler + cache + informers + Prometheus + spot advisor
                                  + AI diagnose + Helm SDK + CRD browser (dynamic client)
                                  Init(clientset, metricsCl, restConfig, clusterName) wires globals.

web/src/
├── main.tsx                    — QueryClientProvider (TanStack Query) wraps App
├── App.tsx                     — router, sidebar, tab switching, WebSocket presence, JIT poll
├── hooks/
│   ├── useFetch.tsx            — original SWR-style fetch hook (still used by most views)
│   └── useApi.ts               — TanStack Query drop-in wrapper (POC; rename useFetch→useApi to migrate)
└── components/
    ├── views/                  — top-level tab views (Overview, Pods, Workloads, CRDs, etc.)
    └── modals/                 — YAML, HelmRelease, Info popover, Settings, Audit, JITRequests, etc.
```

Cross-package callbacks use package-level `var Foo = func(){}` hooks in `notify`, `auth`, `jit`; `main.go` wires them at startup. Never remove or add these without updating `main.go` in the same commit.

Globals in `internal/api/`: `clientset`, `metricsCl`, `restCfg`, `dynamicClient`, `apiextClient`, `cache`. Handlers can assume these are set — `main.go`'s init order guarantees it before `mux` is served.

## The rule: test after every backend change

**Every time you edit `.go` under `internal/` or `cmd/`, run before declaring the task done:**

```bash
cd /Users/manish/Desktop/kube-argus
go build ./... && go vet ./... && go test ./...
```

**Every time you edit `.tsx` or `.ts` under `web/src/`, run before declaring the task done:**

```bash
cd /Users/manish/Desktop/kube-argus/web && npx tsc -b
```

Non-negotiable. A `.claude/settings.local.json` `Stop` hook enforces this — the harness will refuse to end your turn until backend/frontend gates pass on modified files.

If tests fail, do NOT declare done or hand back to the user — fix the failure first.

## Common commands

```bash
# Backend
go build ./...
go vet ./...
go test ./...                              # 24 tests, ~3s
go run ./cmd/server                        # localhost:8080, uses ~/.kube/config

# Frontend
cd web && npm run dev                      # localhost:5173, proxies /api → :8080
cd web && npx tsc -b                       # typecheck
cd web && npm run build                    # production bundle

# Run everything locally (two terminals)
# Terminal 1: aws sso login && go run ./cmd/server
# Terminal 2: cd web && npm run dev
# Open http://localhost:5173/

# Stop servers
lsof -ti:8080 -ti:5173 | xargs kill 2>/dev/null
```

## Coding conventions

**Go:**
- Handlers: `func apiSomething(w http.ResponseWriter, r *http.Request)`. Use `httpx.JSON`, `httpx.Error`, `httpx.K8sError` for responses.
- Admin gate: `if !auth.RequireAdmin(w, r) { return }`. For role-branching without writing 403 use `auth.IsAdmin(r)`.
- Cross-package call: assign to a `var Foo = func(){}` hook in the callee package, wire in `main.go`.
- Audit any admin action or sensitive read: `audit.Record(email, role, action, resource, detail, ip)`. Grab email/role from `r.Context().Value(auth.UserCtxKey).(*auth.SessionData)`.
- New route: register in `internal/api/routes.go` under the right section.
- Multi-verb / sub-path routes: use a dispatcher function (see `helmDispatch`, `apiCronJobRouter`).

**Frontend:**
- Data fetching: prefer `useApi` from `hooks/useApi.ts` (TanStack Query). Legacy `useFetch` still works; migrate incrementally by renaming the import.
- URL state for anything a user should be able to bookmark or navigate back to. Wire popstate in `useEffect` for browser back.
- Tailwind + existing design tokens (`hull-*` for surfaces, `neon-*` for status accents). Don't introduce new palettes.
- New tab: add to BOTH `web/src/routing.ts` `TABS` AND `web/src/components/modals/InfoPopover.tsx` `TABS` + `TAB_DETAIL` — they're duplicated (technical debt worth consolidating one day, but not silently forgetting the second one).

## Security paths that are already handled

Do not re-open these leaks:

| Endpoint | Non-admin behaviour |
|---|---|
| `GET /api/configs/{ns}/{name}?kind=secret` | Values masked as `••••••••`; `masked: true` in response. Uses `auth.IsAdmin(r)`. |
| `GET /api/yaml/Secret/{ns}/{name}` | `.data` and `.stringData` replaced with `***REDACTED-NON-ADMIN***`. Audit-logged as `secret.view`. |
| `GET /api/helm/releases/{ns}/{name}` | Selective heuristic on `config`/`chart.values`, Secret resources in manifest masked, `chart.files` dropped. Audit-logged as `helm.release.view`. |
| `GET /api/helm/releases/{ns}/{name}/revisions/{rev}` | Same redaction as above. |
| `POST /api/helm/releases/{ns}/{name}/rollback` | `auth.RequireAdmin` — viewer gets 403. Audit `helm.rollback`. |
| `DELETE /api/helm/releases/{ns}/{name}` | `auth.RequireAdmin` — viewer gets 403. Audit `helm.uninstall`. |
| `POST /api/workloads/*` (restart, scale) | `auth.RequireAdminOrJIT` — viewer needs an active JIT grant. |
| `POST /api/exec` | Admin or active JIT grant. |
| `POST /api/nodes/*/drain` | Admin only. |

The Helm redaction heuristic: string values under keys containing `password`, `passwd`, `secret`, `token`, `apikey`, `access_key`, `private_key`, `signing_key`, `encryption_key`, `credential`, `bearer`, `dsn` are redacted, EXCEPT when the key ends in `Name`, `Names`, `Ref`, `Refs`, `Reference`, `References` (resource references, not the secret itself). Numbers, booleans, nulls never redacted. Live in `internal/api/helm.go` `isSensitiveLeafKey`.

## Version bump policy

Default to **patch bump** (e.g. `1.2.8` → `1.2.9`) unless the user explicitly asks for a minor/major. Update in this order and NEVER miss a spot:

1. `deploy/helm/kube-argus/Chart.yaml` — `version:` and `appVersion:`
2. `deploy/helm/kube-argus/Chart.yaml` — replace or extend `annotations.artifacthub.io/changes`
3. `CHANGELOG.md` — new section at the top under the `# Changelog` header
4. `docs/index.html` — hero eyebrow pill and OSS section `v1.2.X`

The release workflow `.github/workflows/release.yaml` triggers on any `v*` tag push and handles: multi-arch image build → GHCR push, OCI chart push → GHCR, Helm chart tarball → gh-pages, GitHub Release with CHANGELOG section as body.

Docs landing page auto-syncs from `docs/**` to gh-pages via `.github/workflows/pages-sync.yaml` on push to master.

## Known runtime gotchas

- **Backend does NOT listen until cache is warm.** With expired EKS creds, informers can't sync and the HTTP server never starts. Users see 502 from Vite. Fix path: user refreshes `aws sso login`, then restarts the server.
- **Cache warmup takes 5–15 seconds on real clusters.** Endpoint smoke tests should sleep or poll before hitting endpoints.
- **`/api/audit` is admin-only.** Test scripts asserting 200 need admin auth.
- **Vite dev server proxies `/api`, `/auth`, `/health` to `:8080`.** If backend isn't up, browser sees 502 from Vite proxy.
- **Secret redaction runs recursively.** Any new endpoint that returns a k8s Secret or Helm release object needs the redaction pattern — call `maybeRedact` (Helm) or use the pattern from `internal/api/yaml_editor.go` (raw Secret).

## What NOT to do

- Don't add new admin actions without wiring `auth.RequireAdmin` + `audit.Record`. Every write path must be audited.
- Don't add unregistered handlers. `go vet` won't catch them; they'll silently 404.
- Don't remove the cross-package `var Foo = func(){}` hooks or the wire-up in `main.go`. They're the only glue between packages.
- Don't refactor `internal/api/` into `internal/{k8s,prom,spot,api}/` without a plan. It's tempting; it was the original REORGANIZATION.md target. It's a 1-day focused effort and half-done leaves the codebase worse.
- Don't switch away from ConfigMap persistence for audit / JIT / webhook config unless you're introducing a proper database — the current story is intentional (no external deps, no operator, no CRDs).
- Don't add new npm deps without a clear reason. Current dep count is deliberately small.

## Release process

```bash
# 1. Pre-flight
go build ./... && go vet ./... && go test ./... && cd web && npx tsc -b && cd ..

# 2. Version bump (see policy above)
# 3. CHANGELOG entry
# 4. Chart.yaml annotations

# 5. Commit + tag + push
git add -A
git commit -m "vX.Y.Z: <one-line summary>

<body>"
git tag -a vX.Y.Z -m "vX.Y.Z — <one-line>"
git push origin master
git push origin vX.Y.Z

# 6. Watch release workflow
gh run watch
```

Release workflow publishes: image to GHCR, chart tarball to gh-pages, OCI chart to GHCR, GitHub Release with auto-extracted CHANGELOG body.

## Testing conventions

- Pure functions: standalone `_test.go` in the same package.
- Handler tests: use `httptest.NewRecorder()` + `httptest.NewRequest()`. Mock external clients where needed via fake clientset (`k8s.io/client-go/kubernetes/fake`) — not currently done, but the pattern to reach for.
- Test names: `TestSubject_Case`. Table-driven for enumerable cases (see `TestIsSensitiveLeafKey`).
- Test files sit next to their subject: `internal/audit/audit.go` → `internal/audit/audit_test.go`.
- Prefer testing invariants (dedup, redaction, admin gates) over testing implementation details.

---
> Source: [manishchaudhary101/kube-argus](https://github.com/manishchaudhary101/kube-argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
