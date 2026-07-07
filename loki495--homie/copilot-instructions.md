## homie

> Self-hosted home lab dashboard. Laravel 13 + Livewire 4 (SFC) + Alpine.js + Tailwind v4 +

# Homie — project context

Self-hosted home lab dashboard. Laravel 13 + Livewire 4 (SFC) + Alpine.js + Tailwind v4 +
Flux UI (free tier). SQLite. Dockerized dev environment behind Traefik
(`homie.dev.local.test`).

## UI components

Sidebar manager forms/buttons (Groups, Cards, Discovery) use Flux UI components
(`<flux:input>`, `<flux:select>`, `<flux:textarea>`, `<flux:button>`) — free tier only,
no Pro license. Delete actions use `variant="ghost"` with a `!text-rose-*` class
override (not `variant="danger"`, which renders a filled red button — Flux's own
`text-zinc-800`/`dark:text-white` ghost-variant classes otherwise win the cascade tie
against a plain color override, so the `!` important modifier is required). The
off-canvas sidebar shell and the Groups/Cards/Discovery tab bar remain custom
Alpine — no Flux equivalent was worth the migration risk for either. Dark mode stays
on the project's own `Alpine.store('theme')` + inline FOUC-prevention script; Flux's
`@fluxAppearance`/`@fluxScripts` directives are included for the components' own
needs but the toggle button itself is not Flux's.
- The Cards sidebar list's filter input is plain Alpine (`x-model` + `x-show` per `<li>`
  matching against a `data-search` attribute) — no Livewire round-trip. This is the
  pattern for any future client-side-only filtering: cheaper than a Livewire property
  for something that never needs to touch the server.

## Card icons

`app/Support/DashboardIcons.php` searches the free homarr-labs/dashboard-icons index
(no API key) for icons matching common self-hosted app names, letting card creation
suggest an icon for recognized services. Icons are hotlinked from jsDelivr's CDN —
never downloaded or cached locally on this app's storage (a deliberate choice: keeps
things simple, no storage/cleanup concern, matches how Homarr/Dashy do it). The
`metadata.json` index itself is cached server-side for a day via `Cache::remember`.
`Card.icon` just stores a plain URL — either a resolved CDN link or a manually pasted
one, no distinction made at render time.

## Card API auth

`card_apis.auth_type` selects between `api_key` (sent as an `X-Api-Key` header — the
arr-stack convention) and `basic` (username/password, sent via `Http::withBasicAuth`).
Only one is active at a time based on `auth_type`; the unused fields are nulled out on
save so stale credentials from a previous auth-type choice don't linger. `password` is
encrypted at rest the same way `api_key` already was.

## Provider-specific API stats

`ApiProvider::fetcher()` maps every enum case to an `App\Support\ApiProviders\*Fetcher`
(implementing `ProviderFetcher`) — every provider in the enum has one, non-nullable, by
design (see "Adding a new provider" below). Each fetcher does its own HTTP calls and
returns `{status, summary, stats[], raw}` — `stats` is a small list of label/value pairs
rendered as chips on the card-api-widget instead of the generic "HTTP 200" line.
Endpoint shapes were verified against the gethomepage/homepage widget source (a mature
OSS project with working integrations for all of these) plus live calls against
Andres's own instances — not guessed. Notable per-provider quirks:
- Sonarr: `/api/v3/series` (count), `/api/v3/queue` and `/api/v3/wanted/missing` both
  paginated with a `totalRecords` field — request `pageSize=1` to avoid pulling the
  full list just for the count.
- Radarr v3 has no `/wanted/missing` endpoint (only existed in v1). Missing count is
  computed client-side from `/api/v3/movie`: monitored && !hasFile.
- NZBGet doesn't use an API key — it's HTTP Basic Auth (`ControlUsername`/
  `ControlPassword`), a JSON-RPC POST to `/jsonrpc` with `{"method": "status"}`. This is
  exactly what `auth_type = 'basic'` was built for.
- Prowlarr: same Servarr framework as Sonarr/Radarr, `X-Api-Key` header via
  `ApiHttpClient`. `/api/v1/indexer` (list, filter `enable === true` for the enabled
  count) and `/api/v1/indexerstats` (sum `numberOfGrabs`/`numberOfFailed*` across its
  `indexers` array) — no single endpoint gives an aggregate, so it fetches both.
- Bazarr does **not** follow the arr-stack header convention — it's `?apikey=` as a
  query string only (confirmed: an unauthenticated request to `/api/movies/wanted`
  returns a 401, and gethomepage/homepage's working integration only ever sends the key
  in the query string). `BazarrFetcher` calls `Http::get()` directly instead of going
  through `ApiHttpClient`, since that helper's basic-auth/header logic doesn't apply
  here. Missing-subtitle counts come from `/api/movies/wanted` and
  `/api/episodes/wanted`, both `{"total": N, ...}`.

Adding a new provider: add the enum case, a Fetcher implementing `ProviderFetcher`, and
one `match` arm in `ApiProvider::fetcher()`, all in the same change — the widget needs
no changes. Every case must resolve to a real fetcher (the return type is
non-nullable); don't add an enum case before its fetcher exists.

## Discovery: host-network containers need an inspect fallback

`docker ps`/`/containers/json` report an empty `Ports` for containers running with
`--network host` — there's no mapping to report, the container's ports *are* the host's
ports directly. Without a fallback, every host-network container with no Traefik label
silently vanished from discovery results (found via a real scan: Home Assistant and
ESPHome were both missing from a host with 11 running containers). Both
`discoverViaDocker` and `discoverViaSsh` now do a follow-up lookup for exactly these
stragglers — `docker inspect` (SSH) or `GET /containers/{id}/json` (API) — and read the
first port out of `Config.ExposedPorts` (the image's declared `EXPOSE`, present even
under host networking since it's build-time metadata, not a runtime port mapping).
This only recovers a *port* for containers whose image actually declares `EXPOSE` —
some (Home Assistant, notably) declare none. Rather than drop those silently, they're
still surfaced with a bare `http://{host}` URL (no port) — reachable, we just don't know
at which port, so it's left for the user to fill in when they add the card. Hardcoding a
per-image default port was considered and rejected: it's exactly the kind of
lab-specific special-casing the project's distributability rule forbids (see below), and
there's no reliable app-agnostic source for it. Bridge/default-network containers with
no port and no label are still correctly excluded outright (not surfaced with a bare
URL) — the host-network check (`Networks === 'host'` over SSH,
`HostConfig.NetworkMode === 'host'` via the API) is what gates the fallback, so we don't
invent URLs for containers that genuinely have no path to the host at all.

## API cards are links, but the wrapping lives outside the widget

`card.blade.php` wraps `<livewire:card-api-widget>` in an `<a href="{{ $card->url }}">`
itself, rather than having the widget component decide whether to render a link. This is
deliberate: `$editing` (Arrange mode) needs to gate the link exactly like it already does
for Link-type cards — no `<a>` while arranging, so clicking a card doesn't navigate away
mid-drag. If that logic lived inside `⚡card-api-widget.blade.php` instead, it'd hit the
same lazy-load-island staleness problem as the entry above (`$editing` passed as a prop
would only reflect its value at first mount, not later toggles of Arrange mode). Doing
the wrap in `card.blade.php` sidesteps that entirely — it's a plain Blade partial that
always re-renders fresh with Dashboard, no separate component lifecycle involved.

## Output/API widgets don't reactively see Card edits

`⚡card-output-widget.blade.php` and `⚡card-api-widget.blade.php` are `lazy`-loaded
nested Livewire components, so they're independent islands after their first hydration —
editing a card's name/icon in the sidebar and having Dashboard re-render via the
`dashboard-updated` event does *not* propagate into already-mounted children. (Tried
`#[Reactive]` on the `card` prop first — doesn't cross the lazy-load boundary in
practice, and complicates mount() since a `#[Reactive]` prop can't be reassigned from
inside the component without throwing `CannotMutateReactivePropException`.) Fixed by
giving both widgets their own `#[On('dashboard-updated')]` listener that does
`$this->card = $this->card->fresh();` — cheap, and deliberately does *not* re-run the
shell command / API fetch on every dashboard change.

## Output cards and SSH discovery must never let Process exceptions escape

`⚡card-output-widget.blade.php`'s `mount()` runs a user-supplied shell command
(`Process::timeout(10)->run(...)`) — for remote/SSH commands this can genuinely time
out or fail to spawn, and an uncaught `ProcessTimedOutException` (or any other
`Throwable`) crashes the whole Livewire request with a 500 and a jarring error dialog,
even though the failure is expected/routine (a flaky SSH connection, not a bug). Wrap
every direct `Process::run()` call in try/catch and turn a timeout into a normal
"Command timed out after 10s." card output instead of letting it propagate. The same
applies to `⚡machine-manager.blade.php`'s SSH discovery (`discoverViaSsh` and
`resolveHostNetworkPortsViaSsh`) — both now catch `Throwable` around their
`Process::run()` calls and set `$scanError`/fall back gracefully rather than crashing.
(`ApiHttpClient`-based API fetchers already caught `Throwable` around their HTTP calls
from the start, so they were never affected by this.)

## Discovery: don't gate on published ports alone

Both `discoverViaDocker` and `discoverViaSsh` in `⚡machine-manager.blade.php` check
for a Traefik `Host()` label *before* falling back to requiring a host-published port.
Got this wrong once already (shipped a version that required a published port even
when a Traefik label was present) — silently dropped every container that's only
reachable through Traefik's internal Docker-network routing, which is the common case
when only the reverse proxy itself publishes ports. If touching discovery again, keep
the label-first ordering.

## Design principle: this is a distributable app

No lab-specific machine name, hostname, IP, service, or credential should ever be
hardcoded in application code, config defaults, seeders, or views. Anyone should be
able to clone this repo and get an empty dashboard they configure themselves through
the UI/database — not one pre-wired to Andres's home lab.

- Services, machines, groups, card order, output-card commands, and API connection
  details are all rows in the database, never PHP constants or `.env` values baked
  into a controller.
- `.env`/`.env.example` should only ever contain generic Laravel/app config (DB driver,
  app URL, etc.) — never a specific service's address or token.
- If a feature needs a "default" (e.g. a demo card), seed it behind a flag or a
  separate demo seeder, not the default seeder path.

## Container / infra

- App container: `homie-app` (PHP 8.5-apache). Vite container: `homie-vite`.
- Run PHP tooling via `docker exec -u www-data homie-app ...` — use `-u www-data`
  (not root) so files stay owned by UID 1000, matching the host user on the bind mount.
  Composer script wrappers (`composer pint`, `phpstan`, `rector`, `pest`) already do this.
- SQLite database file lives at `database/database.sqlite`, gitignored (it will hold
  real lab config once the app is used — never commit it).
- `storage/ssh/` is mounted into the container for SSH keys used in "output" card
  commands (arbitrary shell commands the user supplies — it's on the user to make sure
  they run correctly in the container). Gitignored except `.gitkeep`.
- Machine-based SSH discovery keeps its own encrypted private key
  (`machines.ssh_private_key`), decrypted to a 0600 temp file only for the duration of a
  scan — discovery itself never reads from `storage/ssh/`.
- `MachineObserver` (via `MachineSshKeySync`) auto-syncs that same key into
  `storage/ssh/{slug}` (e.g. `storage/ssh/media`) in plaintext, 0600, on every save —
  and deletes it if the key is cleared or the machine is deleted. This is a deliberate
  security tradeoff, confirmed with Andres before building: the DB copy stays encrypted
  and is decrypted only transiently for scans, but the synced copy sits on disk
  permanently (still container-internal, but readable by anyone with filesystem access,
  and it survives container rebuilds since `storage/ssh` is host-mounted) — done to let
  output-card commands SSH to a machine without duplicating key management. If a machine
  is renamed, the old slug's file is left behind harmlessly (not cleaned up — a minor
  known gap, not worth the added complexity to chase).

## Tooling

Default PHPStan level 6, Pint `laravel` preset, Rector `UP_TO_PHP_84` + code quality/dead
code sets (dry-run only — never auto-apply without reviewing the diff). Pre-commit hook
(symlinked from `~/.claude/hooks/laravel-pre-commit.sh`) runs Pint (auto-fix) → PHPStan
(block) → Rector dry-run (block) → Pest (block) on staged PHP files.

## Git

Single-branch: `main`. This project intentionally opts out of the global master/local
branch model (see `~/.claude/CLAUDE.md`) — there's no separate production deployment to
mirror, so work happens directly on `main`. Repo: `loki495/homie` on GitHub (public).

---
> Source: [loki495/homie](https://github.com/loki495/homie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
