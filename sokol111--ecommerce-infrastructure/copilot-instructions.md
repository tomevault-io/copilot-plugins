## ecommerce-infrastructure

> `ecommerce-infrastructure` is the **deployment + shared-CI repo** for the whole platform. It

# AGENTS.md

## What this repo is

`ecommerce-infrastructure` is the **deployment + shared-CI repo** for the whole platform. It
is not a service — it owns no runtime business logic. It holds:

- `helm/` — one thin Helm chart per deployable (5 Go services, 3 Nuxt UIs) plus one shared
  library chart (`shared-helpers`).
- `environments/local/` — the local dev stack: **k3d + Tilt + docker-compose**.
- `environments/production/` — the live **Hetzner k3s** environment: values, raw k8s manifests,
  SOPS secrets, and an ops Makefile driven over an SSH tunnel.
- `docker/` — the Dockerfiles used by **release CI** (`Dockerfile.go`, `.nuxt`, `.seeder`,
  `.logto-seed`). Note there is a *separate* set under `environments/local/docker/` used by
  Tilt for local builds — see **Two sets of Dockerfiles** below.
- `cmd/seeder/` + `cmd/logto-seed/` — two standalone Go programs (each its own module) that
  seed demo data and configure Logto.
- `.github/workflows/` — **reusable** workflows (`workflow_call`) that every service/UI/api repo
  calls. This is the single source of truth for CI/CD across the workspace.

## Component map (this repo)

```
helm/
  shared-helpers/                    library chart (type: library) — all real template logic
  ecommerce-<service>/               thin app chart, depends on shared-helpers via file://
environments/
  local/        Makefile + makefiles/*.mk + Tiltfile + compose/ + values/ + config/
  production/   Makefile + makefiles/*.mk + values/ + k8s/ + grafana/
docker/         release Dockerfiles (used by go-release.yml / nuxt-release.yml / etc.)
cmd/seeder/     demo-data seeder (own go.mod) → ghcr.io/sokol111/ecommerce-seeder
cmd/logto-seed/ Logto bootstrap (own go.mod) → ghcr.io/sokol111/ecommerce-logto-seed
.github/workflows/  reusable CI/CD called by all other repos
```

## Helm: the thin-chart / library-chart pattern

**All template logic lives in `helm/shared-helpers`** (a Helm *library* chart,
`type: library`, version pinned — currently `0.6.0`). Every app chart is deliberately
**thin**: its `templates/*.yaml` are one-line `include` calls into `shared.*` definitions, and
its `values.yaml` supplies the knobs.

```yaml
# helm/ecommerce-catalog-service/templates/deployment.yaml — the whole file
{{ include "shared.deployment" (dict
  "name" (include "template.fullname" .)
  "Labels" (include "template.labels" . | fromYaml)
  ...
) }}
```

Consequences for editing:
- **To change how *all* services are deployed** (probes wiring, config mounting, pod spec,
  labels), edit `helm/shared-helpers/templates/_*.tpl`. This affects every chart at once.
- **To change one service**, edit that chart's `values.yaml` (or the per-env values under
  `environments/*/values/`), not its templates.
- After changing `shared-helpers`, **bump its `version:`** and update the `dependencies` pin
  in each consuming `Chart.yaml`, then `helm dependency update`. Locally, `Tiltfile` runs
  `helm dependency build` automatically (`helm_with_deps`); CI and `make deploy-svc` run
  `helm dependency update` before install.
- Charts vendor the built library under `charts/*.tgz` — this is git-ignored (`.gitignore`),
  rebuilt on demand.

### Values layering
A chart's own `values.yaml` holds safe defaults (image repo, probes, `service.port: 8080`,
ingress disabled). The **real config is layered on per environment**:
- Local: `environments/local/values/<chart>.yaml` (+ `values.debug.yaml` for Go services, which
  wires Delve). The full app config (`mongo`, `kafka`, `security.jwks`, `observability`,
  `tenant.grpc`) is passed as a `config:` YAML block that `shared.deployment` mounts as a
  ConfigMap at `/configs/config.yaml`. Secrets (Logto M2M creds) come via `env` `secretKeyRef`
  into the `logto-credentials` secret.
- Prod: `environments/production/values/<chart>.yaml`.

## Two sets of Dockerfiles (important)

There are two `Dockerfile.go` (and friends) and they are **not** the same:

| Path | Used by | Base build | Notes |
|---|---|---|---|
| `docker/Dockerfile.go` | **release CI** (`go-release.yml`) | pins `go.mod` versions, multi-target `base-release` / `debug-release`, distroless | context = the service repo; `infra` checked out to `infra/` |
| `environments/local/docker/Dockerfile.go` | **Tilt** (local) | reconstructs a `go.work` inside the image | context = workspace root; bundles Delve; ubuntu runtime for debugging |

The local `Dockerfile.go` is the one that implements the "reconstruct `go.work` from local
sources" behavior described in the workspace `AGENTS.md`: Tilt passes `API_DEPS` (a
space-separated list from `GO_SERVICES[...]['api_deps']` in the `Tiltfile`), the Dockerfile
copies each api module out of the build context and `go work use`s it, so **local api changes
flow into the image with no release/tag/bump**. If you add an api dependency to a service,
update its `api_deps` in the `Tiltfile` or the local build will fail with
"API dependency not found".

Release Dockerfiles are consumed by CI via checkout of this repo into `infra/`, so their paths
(`infra/docker/Dockerfile.go`) are load-bearing in `go-release.yml` / `nuxt-release.yml`.

## Local environment (`environments/local/`)

Requires `k3d kubectl tilt helm docker` (`make tools-check`). The Makefile is split into
`makefiles/*.mk` (cluster / docker / infra / tilt / lifecycle). Run everything from
`environments/local/`.

```bash
make init      # one-time: create k3d cluster + inject host CA + install Traefik + Alloy
make dev       # tilt up — starts docker-compose deps AND builds/deploys all services with hot-reload
make up/down   # start/stop the cluster (keeps data)
make clean     # tear everything down (cluster, infra, volumes)
make urls      # print every URL + demo creds
```

`make dev` is all you need for day-to-day work: the `Tiltfile` itself brings up the
docker-compose deps (Mongo, Redpanda, MinIO+imgproxy, Grafana stack, Logto) via
`docker_compose(...)`. `make docker` exists to start those same deps **without** Tilt (e.g.
running a service from your IDE against the local infra); you don't need it before `make dev`.

Layout of the local stack:
- **k3d cluster** (`k3d-cluster.yaml`, context `k3d-dev-cluster`, namespace `dev`). Traefik and
  Grafana Alloy are installed as Helm charts by `make init`/`make up` (`makefiles/infra.mk`).
- **Heavy deps run in docker-compose, not k8s** (`compose/*.yml`), on an external
  `shared-network`. Tilt starts them via `docker_compose(...)` so they show up in the Tilt UI.
- **Services + UIs run in k8s via Tilt**, built from the local Dockerfiles, deployed with the
  Helm charts + local values. Go services expose Delve on `2345` (forwarded to per-service
  ports `2345`–`2352`, see `GO_SERVICES` in the `Tiltfile`).
- Ingress via Traefik at `*.127.0.0.1.nip.io`; Tilt dashboard at `localhost:10350`.
- **Dependency ordering matters**: `logto-seed` (compose) creates the `logto-credentials`
  k8s Secret; the `logto-credentials` `local_resource` waits on it, and every service
  `resource_deps` on it. Don't remove that edge or services start without auth creds.

The **host CA cert** (`certs/ca-certificates.crt`) is copied from `/etc/ssl/certs` at Tilt
startup and baked into images — this is for a corporate TLS-interception proxy. It's git-ignored.

## Production environment (`environments/production/`)

Live target: a single **Hetzner VPS running k3s** (not k3d), namespaces `prod` +
`observability`. Heavy deps are **managed external services** (MongoDB Atlas, Cloudflare R2,
Grafana Cloud via an Alloy DaemonSet); Redpanda, imgproxy, Logto, Postgres, Traefik+cert-manager
run in-cluster. Ops Makefile is split into `makefiles/*.mk` (connectivity / setup / deploy /
status / operations). Run from `environments/production/`.

**kubectl needs an SSH tunnel** — there is no public k3s API endpoint. Two prerequisites, both
one-time-ish: fetch the kubeconfig (`make kubeconfig` → `~/.kube/config-hetzner`) and open the
tunnel (`make tunnel`, forwards local `:6443` → VPS `:6443` over the `hetzner` SSH host alias;
`make tunnel-stop` closes it). Then there are **two ways to talk to the cluster**:

- **Via the Makefile (recommended).** `environments/production/Makefile` does
  `export KUBECONFIG := $(HOME)/.kube/config-hetzner`, so every ops target already points at the
  prod kubeconfig — no env setup needed on your side:
  ```bash
  make kubeconfig   # once: fetch k3s.yaml → ~/.kube/config-hetzner
  make tunnel       # open SSH tunnel to :6443 (host alias "hetzner")
  make status | health | events | logs SVC=catalog-service
  make deploy-svc SVC=catalog-service [TAG=0.1.9]   # manual hotfix deploy (see CD note below)
  make seed TENANT_SLUG=<slug>                       # trigger seeder Job from the CronJob
  make logto-seed                                    # one-time Logto config Job
  ```
- **Via raw `kubectl`/`helm`** (for ad-hoc commands the Makefile doesn't wrap): point
  `KUBECONFIG` at the fetched file yourself. The tunnel must be up either way.
  ```bash
  export KUBECONFIG=~/.kube/config-hetzner   # or: kubectl --kubeconfig ~/.kube/config-hetzner ...
  kubectl get pods -n prod
  ```
  This is the same file the Makefile exports, so both paths hit the same cluster identically.

**Deploys are normally automated CD, not `make deploy-svc`.** A service release triggers the
reusable `.github/workflows/deploy.yml` (`workflow_call`), which checks out *this* repo,
`helm upgrade`s `helm/<service>` into `prod` with `environments/production/values/<service>.yaml`
and `--set image.tag=<release>`. `make deploy-svc` is the manual fallback for hotfixes.

**Secrets are SOPS-encrypted** (`k8s/secrets.enc.yaml`, age key in `.sops.yaml`, only the
`data` field encrypted). Never commit decrypted secrets or run a command that writes them to
disk.
```bash
make secrets-view    # decrypt to stdout only
make secrets-edit    # edit in place (opens VS Code)
make setup-secrets   # sops decrypt | kubectl apply
```

## CI/CD: reusable workflows are the workspace's source of truth

Everything in `.github/workflows/` is `workflow_call`-triggered and **called by the other
repos** — editing them changes CI for the entire workspace, so treat changes as cross-cutting.

- `go-ci.yml` — lint (gofmt + goimports + golangci-lint) + test (`-race`, coverage threshold,
  default 60%, with exclude patterns) + govulncheck. Called by every Go service/api repo.
- `nuxt-ci.yml` — pnpm lint + typecheck + build.
- `go-release.yml` / `nuxt-release.yml` — build image via the **release** Dockerfile here,
  Trivy-scan (fails on HIGH/CRITICAL), push to `ghcr.io/sokol111/<repo>`, tag + GitHub Release.
  Outputs `image_tag`.
- `api-release.yml` — for `-api` repos: buf breaking-change check, verify `gen/` is up to date,
  publish the TS package to GitHub Packages, tag + release.
- `deploy.yml` — the CD entrypoint (see Production above).
- `seeder-release.yml` / `logto-seed.yml` — build & push the two `cmd/` images; `logto-seed.yml`
  also runs the Job against prod.

Images are published to `ghcr.io/sokol111/*`. Versions come from a repo's `VERSION` file (Go)
or `package.json` (Nuxt); release jobs refuse to run if the tag already exists.

## Seeders (`cmd/`)

Each is a **separate Go module** (own `go.mod`), not part of `go.work`.
- `cmd/seeder` — loads `data/*.json` (products, categories, attributes) + `assets/*.jpg`, then
  drives the services' APIs to populate a tenant. Runs as a k8s CronJob defined in the
  **tenant-service** chart (`helm/ecommerce-tenant-service/templates/seeder-cronjob.yaml`);
  trigger a one-off with `make seed TENANT_SLUG=<slug>` (prod) — it does
  `kubectl create job --from=cronjob/...`. Image: `ecommerce-seeder`.
- `cmd/logto-seed` — bootstraps Logto (applications, M2M creds, resources) from `seed.json`,
  writing results into a k8s Secret via client-go. Image: `ecommerce-logto-seed`.

## Conventions & gotchas

- **Commit per-repo.** This repo is independent; a change here that supports a service change is
  a separate commit from the service repo's.
- **Never hand-edit `charts/*.tgz`, `Chart.lock`, or anything git-ignored** — regenerate with
  helm.
- **YAML is 2-space** (`.editorconfig`); Makefiles are tab-indented. Both env Makefiles use
  strict shell flags (`-eu -o pipefail`) and `--warn-undefined-variables` (local) — undefined
  make vars are errors locally.
- When adding a **new deployable**: create `helm/<name>/` (thin chart depending on
  `shared-helpers`), add per-env values under `environments/*/values/<name>.yaml`, register it in
  the `Tiltfile` (`GO_SERVICES` or `UI_SERVICES`, with `api_deps`), and add it to the prod
  `SERVICES` list in `environments/production/makefiles/deploy.mk`.
- `docs/kubectl-cheatsheet.md` has handy prod inspection commands.
```

---
> Source: [Sokol111/ecommerce-infrastructure](https://github.com/Sokol111/ecommerce-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
