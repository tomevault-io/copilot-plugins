## ryra-cli

> Prefer a small, tight, obvious codebase over a large one full of knobs. Every feature, abstraction, error variant, config field, and Step variant is a long-term support burden — it needs docs, tests, migration paths, and has to keep working through every future refactor. Ship less; polish what you ship.

# Ryra Development Guidelines

## Core Principle: Fewer Features, Well Implemented

Prefer a small, tight, obvious codebase over a large one full of knobs. Every feature, abstraction, error variant, config field, and Step variant is a long-term support burden — it needs docs, tests, migration paths, and has to keep working through every future refactor. Ship less; polish what you ship.

Applied:
- Fix the bug in front of you; don't surrounding-cleanup or "while I'm here" refactor.
- Add the smallest thing that works end-to-end. No speculative flexibility, no "might need this later" parameters.
- Prefer extending existing types over introducing parallel ones. Two enum variants < new enum. One new function < new module.
- A symmetric code path in core (planning) is better than a new Step variant when the existing ones already express it (e.g., `Step::WriteFile` for Caddyfile edits — don't add `Step::EditCaddyfile`).
- When in doubt, delete.

## Core Principle: Design for the Right Shape, Not the Smallest Diff

"Fewer features" is about feature surface, not change surface. When the *right* design for a problem requires touching many files, restructuring an enum across multiple crates, or unifying parallel code paths into one, do that — don't ship a narrow patch that leaves the architecture worse and call it done because it's smaller. Design for scalability, not MVP.

A holistic refactor that gets the abstractions right is cheaper long-term than a sequence of minimal patches that each accrete a new special case. If a config field, enum variant, or call site is in the wrong shape for what the system needs to do, fix the shape — even if it ripples through the codebase.

Applied:
- Don't choose a worse design because it's a smaller change. The size of the diff is not the cost; the size of the resulting support burden is.
- Don't preserve a parallel code path "for now" when one unified path is more correct.
- When a new feature exposes that an existing abstraction is wrong, restructure the abstraction. Don't bolt the feature onto the wrong shape and leave the cleanup as future work.
- Design for the system you'll have in a year, not the smallest thing that works today.

This complements "Fewer Features, Well Implemented" — it does not contradict it. Ship less, but make what you ship structurally right. "Smallest thing that works end-to-end" applies to *feature scope* (no speculative knobs, no hypothetical-future parameters); it does not give you license to pick a design you know is wrong because the corrected one is more code.

## Core Principle: Make Invalid State Unrepresentable

Use enums and pattern matching everywhere instead of string comparisons, boolean flags, or if-chains. This applies at every layer:

- **Config values**: DNS, SSL, SMTP, auth providers are enums with associated data, not string fields with optional companions
- **Commands/actions**: Operations returned from core to CLI are typed enums (e.g., `Step::WriteFile { .. }`, `Step::StartService { .. }`), not string commands that get parsed with `.contains()`
- **Service status**: `Available | Installed`, not a bool flag
- **Service kind**: `Application | Infrastructure`, not a string

When adding new functionality, ask: "Can this state be invalid?" If yes, restructure with enums so the type system prevents it. Pattern matching (`match`) must be exhaustive — the compiler enforces that every case is handled.

**Anti-patterns to avoid:**
- `if config.provider == "letsencrypt"` → use `match config.ssl { SslConfig::Letsencrypt { .. } => .. }`
- `if cmd.contains("start")` → use `match step { Step::StartService { .. } => .. }`
- Optional fields that are only valid in certain states → put them inside enum variants

### Validate at the boundary

All TOML files are validated immediately after deserialization — if parsing succeeds, the data is safe to use without further checks:

- **`ServiceDef::validate()`** — duplicate names (ports, env), env var name format, env kind consistency, RAM consistency (recommended >= minimum)
- **`TestToml::validate()`** — mutually exclusive `[[tests]]`/`[[steps]]`, required fields per step action type
- **`Config::validate()`** — no duplicate service names
- **`StepAction`** is an enum, not a string — serde rejects unknown actions at parse time

When adding new fields or service definitions, the compiler and `validate()` should catch structural errors. Never silently default on missing/invalid data — error loudly at load time.

## No Unwraps, No Silent Failures

Never use `.unwrap()`, `.expect()`, or `panic!()`. Every fallible operation must be handled with `?`, `match`, or a meaningful default. This includes:

- `Option` values — use `?`, `ok_or()`, `unwrap_or_default()`, or pattern match
- `Result` values — propagate with `?` or handle explicitly
- Indexing — use `.get()` instead of `[]` where bounds aren't guaranteed

If something truly cannot fail, explain why in a comment and use `unwrap_or_else(|| unreachable!("reason"))` so the reasoning is documented.

**No silent failures:** Never use `.ok()`, `let _ =`, or `unwrap_or_default()` to discard errors that could leave the system in a bad state. Specifically:
- Don't fall back to `/tmp` or empty strings when paths/config can't be resolved — error instead
- Don't silently skip template rendering errors — if auth/SMTP mappings render to empty, that's an error
- Config file permission errors (`chmod 600`) must propagate — security-relevant
- Use `|| true` in shell hooks only when failure is genuinely acceptable, and document why

## Headless / Scriptable by Default

Every `ryra` command must be fully runnable without a TTY (CI, cron, provisioning scripts, the orchestrator). Interactive prompts are a convenience layer over the headless path, never the only way to do something.

- Every value an interactive prompt collects must also be settable via a flag (or process env). If you add a prompt, add the matching flag in the same change.
- Every confirmation must be bypassable with `--yes`. Destructive ones may additionally demand typed confirmation interactively, but `--yes` must still let a script proceed.
- Where it makes sense, offer `--dry-run` to preview without mutating.
- A command run with no TTY and missing required input must fail loudly, naming the flag to pass. Never hang waiting on stdin, and never silently pick a default for something the user should decide. `is_interactive()` (both stdin and stdout are TTYs) gates the prompt path; the non-interactive branch either has every input from flags or errors with a usable message.
- Selection prompts (multi-select, pick-one) need a headless equivalent: a flag to name the targets, or `--yes`/`--all` to take the obvious set. "Apply to everything" via `--yes` is an acceptable headless answer when per-item selection is only an interactive convenience.

## Commits

Follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Prefix every commit subject with a type: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`, `ci:`.

## Releasing

Two steps, both on `main`:

1. Bump the workspace version in `Cargo.toml` (and matching path-dep versions in `crates/ryra-test/Cargo.toml` + `crates/ryra/Cargo.toml`), commit as `chore: vX.Y.Z`, push. CI publishes to crates.io.
2. Tag locally and push from your own identity — *not* from CI: `git tag v0.4.0 && git push origin v0.4.0`. The tag push triggers `build-linux` + `release-version`, which builds .deb/.rpm/.pacman, signs them, creates the GitHub release, and dispatches `deploy-pages` to refresh the apt/rpm/pacman repos.

The split is deliberate: tags pushed by `GITHUB_TOKEN` (i.e. from a workflow) do not trigger downstream workflows, so auto-tagging from `publish-crates` silently skips the build chain. Push the tag yourself.

## Architecture

- `ryra-core`: library with no CLI deps and no sudo. Planning (`ops`, `add_service`, …) is pure: typed results, no system probes. System-touching utilities live under `ryra_core::system` (podman, tailscale, the Step executor `system::apply`) so every frontend shares one implementation
- `ryra` (crate `ryra`, binary `ryra`): thin shell that calls core and handles sudo/system interaction
- `ryra-vm`: QEMU/SSH/cloud-init VM orchestration library
- `ryra-test`: E2E test runner, depends on ryra-vm
- Core returns typed results describing what needs to happen; frontends (CLI, ryra-api) decide whether to apply (via `system::apply`) or print
- All services run under the invoking user's rootless podman (`systemctl --user`)
- Quadlet files go to `~/.config/containers/systemd/`
- Service data goes to `~/.local/share/services/<name>/`
- `service_home()` and `quadlet_dir()` return `Result<PathBuf>` — they error if `$HOME` is unset rather than silently falling back to `/tmp`
- Warns if running as root

## Auth and Caddy Integration

### URL philosophy

Services run at `http://127.0.0.1:<port>` by default — no domain, no HTTPS, no `/etc/hosts` entries needed. The `--url` flag is opt-in for when the user wants to tell ryra where the service will be publicly reachable — whether that's through Caddy, an external reverse proxy, a Cloudflare Tunnel, a Tailscale Funnel, or anything else.

`--url` is modeled as a *fact about the deployment*, not a request to configure routing. Ryra always uses it to populate template variables (`{{service.external_url}}`, `{{service.domain}}`, `{{service.scheme}}`, OIDC callback URLs, email links). Caddy integration is a side-effect that kicks in *when Caddy is installed* — if the user runs their own reverse proxy, they simply don't install Caddy and ryra won't touch routing.

The only exception is **authelia**, which requires a public URL because its OIDC implementation enforces HTTPS for `authelia_url`. When `--auth` is used, authelia auto-installs with a URL (defaults to `https://auth.internal:<caddy_https_port>`). `.internal` is the ICANN-designated TLD (2024) for private networks — unlike `.localhost` it doesn't auto-resolve, so ryra writes `/etc/hosts` entries mapping `*.internal` → 127.0.0.1 during auto-HTTPS promotion (requires sudo once).

This keeps the default experience frictionless (no custom DNS, no public certs) while still supporting custom URLs for production use via Caddy, Tailscale, Cloudflare tunnels, or port forwarding. The one-time `/etc/hosts` edit is the price we pay for a hostname that works cleanly inside containers (Debian-patched libcurl forces `*.localhost` → 127.0.0.1, which broke server-to-server OIDC discovery for PHP-based services like Nextcloud).

### How `--url` works

When `ryra add <service> --url https://foo.example.com` is called:
1. The URL is parsed; `{{service.external_url}}`, `{{service.domain}}` (hostname), and `{{service.scheme}}` are added to the template context
2. If Caddy is installed, a site block is added to the Caddyfile routing `foo.example.com` to the service's port
3. `caddy reload` applies the new route
4. On `ryra remove`, the route is cleaned up (reload is skipped if the Caddyfile becomes empty)

If Caddy is *not* installed, step 2-3 are skipped — ryra populates templates and leaves routing to whatever the user's external setup is.

### How `--auth` works

When `ryra add <service> --auth` is called:
1. An OIDC client ID and secret are generated and injected into the template context (`{{auth.client_id}}`, `{{auth.client_secret}}`, `{{auth.issuer}}`, etc.)
2. Services with native OIDC mappings (`[mappings.auth]` in service.toml) get OIDC env vars written to `.env`
3. Native OIDC services join the auth provider's podman network for direct HTTP communication
4. OIDC client registration is handled by `authelia.rs` (edits authelia's configuration.yml)
5. Quadlet `ExecStartPost=` scripts handle runtime OIDC setup (API calls, config injection) after the container starts
6. Services without native OIDC get Caddy forward auth instead (Authelia handles login at the proxy level)
7. Services work at `http://127.0.0.1:<port>` without `--url` — the OIDC redirect goes through authelia (HTTPS) and back to the service (HTTP)

### How metrics work

Prometheus (`provides = ["metrics-store"]`) and grafana (`provides = ["metrics-dashboard"]`) wire up through the capability system, both install orders:

1. A service declares its endpoint in service.toml: `[metrics]` with `port` (a `[[ports]]` name; the scrape target uses its *container* port) and optional `path` (default `/metrics`).
2. When a store is installed, adding a `[metrics]` service drops `targets/<svc>.json` into the store's file_sd dir (live, no reload) and joins the service to the store's network. `ryra remove` deletes the target file.
3. Adding a dashboard provider with a store installed provisions a datasource file into `provisioning-datasources/` (read at boot). Installing the store *after* other services wires everything retroactively (`retroactive_metrics_wiring`): targets, datasources, network joins, restarts.
4. All glue is file-based and typed Steps from `metrics_bridge.rs` - core never calls prometheus or grafana APIs.

### Pre-start and post-start scripts

Hooks are implemented as **quadlet-native `ExecStartPre=` / `ExecStartPost=`** directives in `.container` files — there is no hook abstraction in service.toml. Scripts live in `registry/<service>/configs/scripts/` and are copied to the service's data directory during `ryra add`. The quadlet file references them with `ExecStartPost=/bin/bash ${SERVICE_HOME}/configs/scripts/<script>.sh`.

The service's `.env` file is loaded by the quadlet `EnvironmentFile=` directive, so env vars like `$SERVICE_PORT_HTTP`, `$OAUTH_CLIENT_ID`, etc. are available to ExecStartPost scripts. Scripts access bind-mounted volumes via `$SERVICE_HOME` (pointing to `~/.local/share/services/<service>/`).

### Template variables for auth

Available when `--auth` is used and an auth provider (authelia) is installed:
- `{{auth.url}}` — external URL of the auth provider
- `{{auth.internal_url}}` — how containers reach the auth provider directly via HTTP (`http://systemd-authelia:<port>` on shared podman network)
- `{{auth.issuer}}` — OIDC issuer URL (provider-specific)
- `{{auth.client_id}}` — generated UUID for OIDC client
- `{{auth.client_secret}}` — generated 64-char secret for OIDC client
- `{{auth.provider}}` — provider name (e.g., "authelia")

## Service Configuration Philosophy

Prefer environment variables and declarative config for all service setup. When a service can't be fully configured through envs alone (e.g., it requires plugin installation, API calls, or config file generation), use quadlet `ExecStartPre=` / `ExecStartPost=` scripts to automate those steps. The goal is that `ryra add <service>` is turnkey — the user shouldn't need to manually configure the service afterward. If some manual steps are truly unavoidable (e.g., initial web wizard, recommended dashboard imports), put them in `post_install` under `[service]` — printed once as "Next steps:" after a successful `ryra add`.

## Podman & Quadlet-Native Solutions

Always prefer podman-native and quadlet-native features over workarounds:

- **Networking**: Use podman networks for cross-container DNS resolution instead of `AddHost` with hardcoded IPs. Services with `--auth` join the auth provider's network for direct HTTP communication.
- **Service discovery**: Containers on the same network can resolve each other by container name (e.g., `systemd-authelia`). Use this instead of `host.containers.internal` or IP addresses
- **Volumes**: Bind-mount everything under `~/.local/share/services/<svc>/<role>/` (db-data, data, media, etc.). Don't ship `.volume` files — named volumes hide state in podman's namespace and split the user's backup target. Use `Volume=${SERVICE_HOME}/<role>:/container/path:Z,U`.
- **Dependencies**: Use `After=` and `Requires=` in quadlet `[Unit]` sections for startup ordering
- **Health checks**: Use quadlet `HealthCmd=` instead of custom wait scripts

### Quadlets are plain podman files

Registry quadlets are NOT templates. Ryra uses them exactly as authored (plus additive injections: networks, `--auth` volumes, ExecStartPre). They work without ryra too: copy the `.container` file into `~/.config/containers/systemd/` and write the `.env` by hand. Requires podman >= 5.3 (quadlet passes port values through to the podman command line instead of validating them; systemd then expands `${...}` in the generated `ExecStart=` from the `[Service]` section's `EnvironmentFile=`).

The contract:

- `${SERVICE_PORT_<NAME>}` — resolved host port for the `[[ports]]` entry `<name>`, from the `.env`. Works in `PublishPort=`, `Volume=`, `ExecStartPre/Post=`. Core validates every reference against declared ports at add time (an undefined var would silently expand to "" at runtime).
- `${SERVICE_HOME}` — the service's data dir (`~/.local/share/services/<svc>`), from the `.env`. Use it for all bind mounts and script paths.
- `EnvironmentFile=%h/.local/share/services/<svc>/.env` — the one literal path in the unit. Systemd resolves `EnvironmentFile=` before any env exists, so it cannot be env-based; `%h` is a native systemd specifier. Both `[Container]` (container env) and `[Service]` (expansion + ExecStartPre/Post env) need the line.
- `${POSTGRES_USER}`-style vars in `HealthCmd=` expand inside the container at runtime, as before.

Never add install-time rewrites of quadlet content in core. The single exception: when the resolved data root differs from the canonical `~/.local/share/services` (the `RYRA_DATA_DIR` test sandbox, or a custom `XDG_DATA_HOME`), core repoints the `EnvironmentFile=` path so the unit reads the `.env` ryra actually wrote. Default setups use the file byte-for-byte (plus provenance header and injections).

## System Dependencies

- `podman` — rootless containers for services
- `systemd` — user-level service management (`systemctl --user`)

## Debugging

**Prefer iterative checks over static timeouts.** When waiting for something to become ready (a service to come up, a port to bind, a file to appear, a health endpoint to answer), poll the actual condition in a loop instead of sleeping a fixed amount and hoping. The loop should:
- **check the real readiness signal**, not a proxy — e.g. probe the container's health (`podman healthcheck run`) or hit the endpoint, not just "the systemd unit went active";
- **tell the user what it's doing** while it waits — what it's checking, that it's still checking, and the attempt count — so a long wait never looks like a hang;
- **exit the instant the condition is met**, so fast machines aren't penalized and slow machines still succeed.
A static timeout is the *bound* on the loop, not the wait itself. Reserve a genuinely long static timeout for the rare case where there's no cheaper signal to poll (and say why in a comment).

**Never increase timeouts before identifying the root cause.** When a test times out, check logs first to understand *why* it's slow — don't just bump the number. Only increase a timeout after you've confirmed the issue is genuinely timing-related (e.g., a heavy image pull or slow SPA hydration) and there's no underlying bug. Timeouts mask real issues and make tests slow.

When tests fail or services error, **always check logs first** before proposing fixes:
- `journalctl --user -u <service>.service` for service logs
- `podman logs <container>` for container output (includes application logs)
- `podman exec <container> cat /path/to/app.log` for application-specific logs (e.g., seahub.log)
- `podman ps -a` to see container state
- `ss -tlnp` to check port bindings

This applies during development too — when a service fails after `ryra add`, check the logs to find the root cause rather than guessing.

## E2E Testing

E2E coverage is scoped to flows that cross service boundaries — primarily SMTP delivery and OIDC login. Services without those integrations (e.g. Synapse, Vaultwarden when added without `--auth`) don't need a dedicated E2E test; a simple install assertion is enough.

### Run modes

`ryra test` runs against **this host by default** — there is no implicit VM. The mode is selected by flags on `ryra test` itself:

- **default** (no flag): full `add` → assert → `remove`/purge lifecycle **on this host**. It installs, purges, and reinstalls each service the test declares, and runs arbitrary shell/HTTP commands from the registry against the real machine. Unrelated services and their data are left untouched. Mutating — gated by `confirm_host_run`: an interactive prompt, or a hard refusal in non-interactive shells unless `-y` is passed.
- `--vm`: run the same full lifecycle inside a fresh, throwaway QEMU VM instead of on the host. Slower, needs KVM, but isolated — the host is never touched. This is what CI uses.
- `--live`: run every check test's `[[tests.checks]]` flow against the services **actually installed** on this host — the setup bracket is skipped, assertions run against the real `.env` and ports, no sandbox, nothing installed or removed. Services not installed are skipped. Exits non-zero on any failure, so a systemd timer running `ryra test --live -y` doubles as behavioral monitoring. This is `--live`'s only behavior; there is no single-service variant.

`--vm` answers *where* (disposable VM vs. host); `--live` answers *what* (checks only vs. setup + checks). Having a `[[tests.checks]]` flow is what makes a test live-capable; the same flow also runs non-live (sandbox/VM/CI) after its setup bracket.

Tests are selected by the positional `NAMES` argument (or `--all`) in every mode — that is the one filter.

Key points:

- `--distro=debian-13` (default) or `--distro=fedora-43` selects the VM base image (flags on the test runner binary, not `ryra test`; only relevant under `--vm`/`--keep-alive`)
- Test runner lives in `crates/ryra-test/`, VM orchestration in `crates/ryra-vm/`
- Tests are defined in `registry/` via `[[tests]]` in service.toml and lifecycle test files in `registry/tests/`
- Under `--vm`, VMs use cloud images + cloud-init for setup, SSH for command execution, get a fresh Linux install with their own kernel, and VM memory is auto-sized per test based on `[requirements.ram]` in each service's service.toml
- `--parallel=N` controls concurrency (default 1), each VM gets a unique SSH port
- KVM is required for reasonable speed (`--no-kvm` works but is ~10x slower, flag on test runner binary)
- `--keep-alive` boots a VM and keeps it running after tests for interactive debugging (inherently a `--vm` operation)
- `--verbose` dumps the serial log on failure
- VM host prerequisites (Debian/Ubuntu): `qemu-system-arm qemu-utils qemu-efi-aarch64 genisoimage openssh-client curl`
- VM host prerequisites (Fedora): `qemu-system-aarch64 qemu-img edk2-aarch64 genisoimage openssh-clients curl`

### Test types

Every test is a typed step sequence in a `test.toml` (the legacy `[setup]` +
`run` shell format is rejected at parse time with a migration pointer):

- **Lifecycle tests** (`[[tests.steps]]`): interleaved add/remove/wait/shell/
  http/playwright/mail steps. Service-owned files live in
  `registry/<service>/test.toml`; cross-cutting ones in `registry/tests/<name>.toml`.
- **Check tests** (`[[tests.checks]]`, optional `[[tests.setup]]` bracket):
  the dual-mode shape; see "Check tests (setup + checks)" below.

### OIDC lifecycle tests

OIDC tests must install caddy and authelia before adding services with `--auth --url`. The test steps are:
1. `ryra add caddy` — reverse proxy
2. `ryra add authelia --url https://auth.test.local` — OIDC provider
3. `ryra add <service> --auth --url https://<service>.test.local` — service with OIDC
4. Assertions verify HTTP responds and OIDC is configured

### Adding a new service

After creating a service definition (service.toml, quadlets, configs), always run the E2E tests to verify it works. Use `--keep-alive` extensively to iterate — it boots the VM once and keeps it running so you can SSH in, inspect logs, check the UI, and fix issues without waiting for a fresh boot each time.

**Before writing a new test, skim existing ones with `ryra test search -v [name-filter]`.** The verbose listing shows every step (ryra subcommand, HTTP probes, shell bodies, mail polls, playwright env) inline — it's the fastest way to learn the conventions for basic install / `--smtp=inbucket` / `--auth` flows without opening a bunch of `.toml` files.

Tests for a service all live in `registry/<service>/test.toml` as a `[[tests]]` array, optionally split across extra `registry/<service>/*.test.toml` files (e.g. `checks.test.toml`, `backup.test.toml`) which use the same format and naming. Convention: the first test is named after the service (so `ryra test <svc>` runs only it); additional tests are named `smtp`, `oidc`, `diff`, etc., and get prefixed automatically (`<svc>-smtp`, `<svc>-oidc`). Cross-cutting multi-service tests live in `registry/tests/*.toml`.

### Check tests (setup + checks)

A `[[tests]]` entry has exactly one body: legacy `run`, lifecycle `steps`, or `checks`. A test with **`[[tests.checks]]`** is a check test: the checks are the live-runnable flow (`ryra test --live` runs exactly them against the real installs), and sandbox/VM/CI runs execute setup + checks — so the same definition is both the install test and the live check.

- **Structure is the safety, not a flag.** The `checks` step schema has no `add`/`remove` (checks must be safe against live installs) and no `mail` (it reads the inbucket fixture, which live installs don't have — SMTP assertions stay in lifecycle `steps`). Setup only admits `add`/`wait`. Both are enforced at parse time with pointed errors.
- **Setup defaults to the owning service.** Most service test.tomls write only `[[tests.checks]]`; discovery brackets them with `add <svc>` + `wait <svc>`. Write `[[tests.setup]]` explicitly when the install needs args (`--smtp=inbucket`, `--auth --url …`) or multiple services (each `add` counts; live mode requires them all installed, else skips). Cross-cutting files under `registry/tests/` have no owner, so they need explicit setup or `services = ["a", "b"]` (which synthesizes add+wait per service; `services` + explicit setup together is a parse error).
- **The first rule of writing checks: read the install, never assume the setup.** Live checks run against installs configured by the user, not by your setup args — derive every value from the service's real `.env` (via `${RYRA_DATA_DIR:-$HOME/.local/share/services}/<svc>/.env`, which resolves to the sandbox in CI and the real dir live). `http` steps without `service = "…"` get it defaulted when the test has one service and are a parse error otherwise (the source-every-.env fallback would read unrelated installs live).
- **Env, by layer**: install-time env goes on the setup `add` step (`env = { … }`) and only affects sandbox installs — live checks see the real install's values via its `.env`. Per-step env goes on shell (inline in `run`) or playwright (`env = { … }`) steps and applies in both modes. Test-level `env` is legacy-`run`-only and is rejected elsewhere.
- **What stays a lifecycle test (`steps`)**: remove/teardown tests, upgrade/diff tests, and mail-asserting SMTP flows. `wait` inside `checks` is allowed and runs live as an "unit is active" assertion.
- Exemplar: `registry/grafana/test.toml` (checks with defaulted setup; the `smtp` sibling stays lifecycle).

1. Start with `--keep-alive` to validate the service starts:
   - Boot the VM: `ryra test <service> --keep-alive --yes`
   - SSH in and check logs: `journalctl --user -u <service>.service`, `podman logs systemd-<service>`
   - Verify the service is actually responding before writing assertions
2. Add a basic install test as the first `[[tests]]` entry in `registry/<service>/test.toml`. Run `ryra test <service>` to verify.
3. If the service has OIDC integration (`auth = ["oidc"]` in service.toml), add an `oidc` test entry with `browser = true`:
   - Create `registry/tests/browser/<service>-auth.spec.ts` with Playwright tests
   - Use `--keep-alive` on `ryra test <service>-oidc` to boot the VM with caddy + authelia + the service, then SSH in and use `curl` to inspect the actual login page HTML — find the real CSS selectors, button text, and post-login indicators before writing the Playwright spec
   - The browser test must click the SSO button, authenticate with Authelia (fill username/password, submit, handle consent), and verify the redirect back results in an authenticated session
   - Don't guess at selectors — look at the real page. Iterate with `--keep-alive` until the test passes
4. If the service sends mail, add an `smtp` test using the `mail` step to assert inbucket delivery (see `registry/forgejo/test.toml` or `ryra test search -v forgejo-smtp`).

When E2E tests have long setup times (pulling images, waiting for services), don't just wait — check logs periodically. If a service takes more than 60s to become ready, SSH in via `--keep-alive` and investigate rather than increasing timeouts blindly.

---
> Source: [ryanravn/ryra-cli](https://github.com/ryanravn/ryra-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
