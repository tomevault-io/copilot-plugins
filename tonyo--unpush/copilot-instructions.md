## unpush

> This document helps AI assistants understand the codebase and contribute effectively.

# CLAUDE.md - unpush

This document helps AI assistants understand the codebase and contribute effectively.

## Project overview

`unpush` is a continuous deployment service for [Uncloud](https://github.com/psviderski/uncloud). It runs as a container inside an Uncloud cluster and deploys services automatically when a branch changes, triggered by GitHub push webhooks or by polling the remote branch on a configurable interval.

The deployer connects to the Uncloud daemon through its Unix socket (`/run/uncloud/uncloud.sock`), which every node in the cluster exposes. This gives the deployer full cluster access without needing SSH keys or network configuration.

## Repository structure

```
main.go         Entry point. Loads config, opens state DB, registers webhook routes, runs the HTTP server.
config.go       Loads AppConfig from a YAML file (UNPUSH_CONFIG, default /deploy/config.yaml).
webhook.go      GitHub webhook handler. Reads body, verifies HMAC, dispatches to deployer.
deployer.go     Core deploy logic. Connects to socket, loads compose file, plans and executes deploy.
build.go        Builds services with a build directive and pushes images to cluster machines.
repo.go         Clones or fetches the repository at the push commit (repo mode only).
state.go        SQLite state database. Records every deploy attempt and outcome per target.
poller.go       Poll trigger. Checks remote HEAD on an interval and retries failed deploys.
Dockerfile      Multi-stage build. Build context is the deployer directory.
compose.yaml    Reference compose file for deploying the deployer itself into Uncloud.
mise.toml       Build tooling. Go 1.26.1 via mise. Tasks: build, run, build:image.
misc/design.md  Architecture decisions and options considered during design.
```

## Key types

`AppConfig` — top-level config with `ListenAddr`, `StateDB`, and `Targets []TargetConfig`.

`TargetConfig` — per-target settings: `Name`, `WebhookSecret`, `Branch`, `ComposeFile`, `ForceRecreate`, `RepoURL`, `RepoToken`, `WorkDir`, `PollInterval`, `EnableWebhook`, `SocketPath`. Each `Deployer` holds one `TargetConfig`.

`Deployer` — holds a `TargetConfig`, a shared `*sql.DB`, and a buffered channel queue (capacity 1).

## Configuration

Configuration is always loaded from a YAML file. `loadAppConfig` reads the path from `UNPUSH_CONFIG`, defaulting to `/deploy/config.yaml`. `loadFileConfig` parses the file and fills in defaults: `branch` → `main`, `work_dir` → `/deploy/work/<name>`, `compose_file` → `compose.yaml` if `repo_url` is set, `/deploy/compose.yaml` otherwise, `state_db` → `/deploy/state.db`, `enable_webhook` → `true`. It also validates that every target has a unique non-empty name and copies the global `socket_path` into each `TargetConfig`.

Each target registers a webhook handler at `/webhook/<name>` by default. Set `enable_webhook: false` to skip registration (requires `poll_interval`). Poll and webhook triggers can be active simultaneously on the same target.

## Key dependencies

The deployer imports `github.com/psviderski/uncloud` as a standard Go module dependency pinned to a specific commit. The uncloud packages used directly:

| Package                | Purpose                                                                   |
| ---------------------- | ------------------------------------------------------------------------- |
| `pkg/client`           | `client.New` creates a cluster client from a connector                    |
| `pkg/client/connector` | `NewUnixConnector` connects to the daemon socket                          |
| `pkg/client/compose`   | `LoadProject` and `NewDeploymentWithStrategy` implement `uc deploy` logic |
| `pkg/client/deploy`    | `RollingStrategy` controls how containers are updated                     |

In repo mode, the deployer also uses these directly (both are transitive dependencies of uncloud):

| Package                                            | Purpose                                                      |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `github.com/docker/cli/cli/command`                | Creates a Docker CLI client for the build step               |
| `github.com/docker/compose/v2/pkg/compose`         | Builds images via the Compose Go library                     |
| `github.com/google/go-containerregistry/pkg/crane` | Pushes images to remote machine unregistries over plain HTTP |
| `modernc.org/sqlite`                               | Pure-Go SQLite driver (no cgo) for the deploy state database |

`internal/cli.BuildServices` in uncloud contains equivalent build logic but is not importable from outside the module. `build.go` replicates the relevant parts. See the TODO comment there for the long-term option.

`client.PushImage` in uncloud is not used for pushing because it relies on a proxy mechanism that requires the pushing process and the Docker daemon to share a network namespace. When unpush runs as a container, the Go process and the Docker daemon are in different network namespaces, so the proxy on `127.0.0.1` is unreachable from the daemon. Instead, `build.go` pushes directly from the Go process to each remote machine's unregistry using crane over plain HTTP. The WireGuard routing that Uncloud sets up allows the container to reach `machineIP:5000` on remote nodes directly.

## Deploy flow

Both triggers can be active on the same target simultaneously. Each target always has a deploy loop goroutine; the triggers share the same buffered channel queue (capacity 1).

**Webhook trigger** (active when `enable_webhook` is true, which is the default):

1. `webhook.go` receives POST `/webhook/<name>`.
2. HMAC signature is verified against the target's `WebhookSecret`.
3. The event is checked: must be a push to the configured branch.
4. `triggerDeploy` sends the event to the deployer's channel. A second concurrent event is queued. A third is dropped with a warning.
5. `deployLoop` runs in a goroutine and processes events one at a time.

**Poll trigger** (active when `poll_interval` is set):

1. `startPoller` runs in a goroutine. It seeds `lastCommit` from the state DB (most recent deploy record for that target) and falls back to the local git HEAD if no record exists.
2. It calls `poll` immediately on startup, then on each interval tick.
3. `poll` fetches the remote HEAD via `git ls-remote`. If the commit changed, it calls `triggerDeploy`. If the commit is the same but the last deploy record for that commit shows failure, it retries by calling `triggerDeploy` again.

**Deploy execution (`runDeploy`):**

1. If `RepoURL` is set: clones or fetches the repository, checks out the exact push commit.
2. Connects to the Uncloud socket and loads the compose file.
3. If `RepoURL` is set and any services have a `build` directive: builds images locally via Docker, then pushes them to each remote cluster machine directly over WireGuard. The local machine is skipped because the image is already in its containerd store.
4. Plans and executes the deployment.
5. On completion (success or any failure): writes a row to the `deploys` table in the state DB with `succeeded = true/false`.

## Development

```bash
# Install tools
mise install

# Build binary
mise run build

# Build Docker image
mise run build:image
```

## Testing

Unit tests cover config loading, HMAC signature verification, and webhook routing. Run them with:

```bash
go test ./...
```

To test the webhook handler manually, start the server with a config file:

```bash
cat > /tmp/unpush-config.yaml <<'EOF'
targets:
  - name: app
    webhook_secret: test
    branch: main
EOF
UNPUSH_CONFIG=/tmp/unpush-config.yaml mise run run &

curl -X POST http://localhost:8080/webhook/app \
  -H "X-GitHub-Event: push" \
  -H "X-Hub-Signature-256: sha256=$(echo -n '{"ref":"refs/heads/main","repository":{"full_name":"you/app"},"head_commit":{"id":"abc12345","message":"test"}}' | openssl dgst -sha256 -hmac test | awk '{print $2}')" \
  -H "Content-Type: application/json" \
  -d '{"ref":"refs/heads/main","repository":{"full_name":"you/app"},"head_commit":{"id":"abc12345","message":"test"}}'
```

## CI and release

CI runs on every push to `main`, `test/**`, and `release/**`, and on pull requests to `main`. It runs lint (`golangci-lint`) and tests (`go test ./...`) in parallel jobs.

To release a new version, push a `v*` tag:

```bash
git tag v1.2.3
git push origin v1.2.3
```

The CI runs lint and tests first. If they pass, it builds the Docker image and pushes it to `ghcr.io/tonyo/unpush` with three tags: `:1.2.3`, `:1.2`, and `:latest`.

## Documentation style

Follow the same conventions as `uncloud`:

- Use conversational language, write as if speaking to a colleague.
- Keep sentences simple and optimise for clarity.
- Do not use em dashes or semicolons. Break complex sentences into simpler ones.
- Place the subject before the action. Prefer "The deployer loads the compose file" over "The compose file is loaded by the deployer."

---
> Source: [tonyo/unpush](https://github.com/tonyo/unpush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
