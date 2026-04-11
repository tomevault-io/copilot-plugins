## prometheus-site

> This monorepo ships as a reusable web showcase template: a Qwik frontend that streams binary-rendered fragments from a combined Rust axum + WebTransport runtime, plus an Electrobun desktop shell that can package the hosted app for desktop distribution. The default branding currently renders as **Fragment Prime** on the **Prometheus** hostnames, but template defaults now live in `packages/template-config/src/index.ts` and can be rewritten with `bun run template:init`. Keep the primary template surface focused on `apps/{site,desktop}`, `packages/{core,platform,platform-rs,ui,template-config,spacetimedb-client}`, `scripts`, `infra`, `tests`, and `docs`. Advanced or internal subsystems live under `extras/`.

# AGENTS

This monorepo ships as a reusable web showcase template: a Qwik frontend that streams binary-rendered fragments from a combined Rust axum + WebTransport runtime, plus an Electrobun desktop shell that can package the hosted app for desktop distribution. The default branding currently renders as **Fragment Prime** on the **Prometheus** hostnames, but template defaults now live in `packages/template-config/src/index.ts` and can be rewritten with `bun run template:init`. Keep the primary template surface focused on `apps/{site,desktop}`, `packages/{core,platform,platform-rs,ui,template-config,spacetimedb-client}`, `scripts`, `infra`, `tests`, and `docs`. Advanced or internal subsystems live under `extras/`.

## Architecture overview

- **Workspaces:** Managed with Bun (`bun@1.3.5`). Site entrypoint lives in `apps/site`, the desktop shell lives in `apps/desktop`, and the combined API/WebTransport entrypoint lives in `packages/platform-rs/src/main.rs`. Core, platform, platform-rs, UI, template-config, and the shared SpacetimeDB client live under `packages/`.
- **Template docs:** `docs/template-reference.md` is generated from the bundle manifest, and `docs/template-maintainer-guide.md` is the maintainer guide for presets, branding, bundle ownership, and template checks.
- **Core (`packages/core`):** Fragment planning/rendering, binary codecs, client streaming helpers, fragment registry, and prefetch/speculation utilities.
- **Platform/runtime (`packages/platform-rs`):** Rust axum + WebTransport runtime, embedded Yjs signaling, env/config resolution, SpaceTimeDB/Garnet clients, rate limiting, and bundle-aware API route composition.
- **UI (`packages/ui`):** Design system (global styles, RouteMotion, Dock, FragmentCard, toggles), no data fetching.
- **Features (`apps/site/src/features + packages/platform-rs/src`):** Auth, Store, Messaging, Lab (self-contained front/back logic).
- **Site (`apps/site`):** Qwik + Qwik City SPA/SSR composition layer, FragmentShell, routes, branding/copy.
- **API/runtime (platform entrypoint):** Combined Rust entrypoint that boots the platform server, registers site fragment definitions, serves WebTransport, and hosts the `/yjs` signaling endpoint from the same runtime.
- **Infrastructure (`infra/` + `docker-compose.yml`):**
  - Caddy terminates TLS and routes the active site host for the current runtime mode to the web/API containers.
  - Caddy serves HTTP over TCP (h1/h2); UDP 4444 is bound to the integrated WebTransport runtime for HTTP/3 sessions.
  - SpaceTimeDB 2.0 + Microsoft Garnet containers with healthchecks; SpaceTimeDB uses a persistent volume and Garnet runs as an in-memory low-latency cache tier.
  - Dynamic Caddy config generated for dev via `scripts/compose-utils.ts` (writes `infra/caddy/Caddyfile`, controlled by `scripts/runtime-config.ts`).

## Dev and runtime flow

- **Local dev entrypoint:** `bun run dev` (runs Compose services, ensures the Caddy file, starts Qwik dev server on 4173 with HTTPS routed through Caddy at `https://prometheus.dev`, disables `https://prometheus.prod`, and defaults to the `full` template preset).
- **Preview entrypoint:** `bun run preview` (builds the site for the prod host set, serves `https://prometheus.prod` through Compose/Caddy, and disables `https://prometheus.dev`).
- **Runtime defaults:** canonical host/port/profile/preset defaults live in `scripts/runtime-config.ts`.
- **Storybook:** Storybook stays scoped to `apps/site`. Use the app-local scripts inside `apps/site` when you need it; keep the repo root limited to template-facing commands.
- **Desktop shell:** `apps/desktop` packages the existing HTTPS app through Electrobun. It defaults to `https://prometheus.dev` for dev builds and `https://prometheus.prod` for canary/stable builds, and you can override the target with `PROMETHEUS_DESKTOP_TARGET_URL` or the per-channel `PROMETHEUS_DESKTOP_TARGET_URL_{DEV,CANARY,STABLE}` envs.
- **Fragment HMR (dev):** Vite watches `apps/site/src/fragment`, `apps/site/src/fragment/definitions`, and fragment island components under `apps/site/src/components`, emits `fragments:refresh`, clears in-memory/local fragment shell + plan cache on refresh, and re-fetches fragment payloads with `refresh=1` (dev-only); requires the API running from source (dev/watch) and plan changes still require a reload.
- **Build:** `bun run build` builds the default `full` preset; `bun run build:core` builds the lean preset.
- **Template presets:** `PROMETHEUS_TEMPLATE_PRESET`, `TEMPLATE_PRESET`, and `VITE_TEMPLATE_PRESET` resolve the shared feature bundle set. `full` is the default showcase preset; `core` keeps the lean web shell (`auth`, `account`, `demo-home`). Override bundles with `PROMETHEUS_TEMPLATE_FEATURES` / `PROMETHEUS_TEMPLATE_DISABLE_FEATURES` (or the `TEMPLATE_` / `VITE_TEMPLATE_` variants) using comma/newline-separated feature ids.
- **Bundle-owned runtime defaults:** realtime-owned envs default on only when the `realtime` bundle is enabled, analytics/highlight/Partytown envs only matter when `analytics` is enabled, service-worker/PWA behavior only activates when `pwa` is enabled, and `native` remains off unless explicitly enabled.
- **Fragment cache TTLs:** `FRAGMENT_PLAN_TTL` (seconds, default `180`), `FRAGMENT_PLAN_STALE_TTL` (seconds, default `300`), and `FRAGMENT_INITIAL_TTL` (seconds, default `180`) control fragment plan + initial payload cache lifetimes.
- **WebTransport envs:** `WEBTRANSPORT_API_BASE` (defaults to `http://api:4000`), `WEBTRANSPORT_LISTEN_ADDR` (defaults to `:4444`), `WEBTRANSPORT_CERT_PATH`, `WEBTRANSPORT_KEY_PATH`, `WEBTRANSPORT_ALLOWED_ORIGINS`, `WEBTRANSPORT_ALLOW_ANY_ORIGIN`, `WEBTRANSPORT_ENABLE_DATAGRAMS` (defaults to on), `WEBTRANSPORT_MAX_DATAGRAM_SIZE` (defaults to `1200`), `PROMETHEUS_WEBTRANSPORT_PORT` (defaults to `4444` for host UDP), `VITE_WEBTRANSPORT_BASE` (optional client override).
- **P2P relay + ICE envs:** `VITE_P2P_RELAY_BASES`/`P2P_RELAY_BASES` (comma/newline list of API bases for mailbox relays; defaults to resolved API base), `VITE_P2P_NOSTR_RELAYS`/`P2P_NOSTR_RELAYS` (comma/newline list of `ws(s)` Nostr relays), `VITE_P2P_WAKU_RELAYS`/`P2P_WAKU_RELAYS` (comma/newline list of Waku bootstrap multiaddrs, optionally prefixed with `waku:`), `VITE_P2P_CRDT_SIGNALING`/`P2P_CRDT_SIGNALING` (comma/newline list of y-webrtc signaling URLs), `VITE_P2P_PEERJS_SERVER`/`P2P_PEERJS_SERVER` (PeerJS server URL), and `VITE_P2P_ICE_SERVERS`/`P2P_ICE_SERVERS` (JSON array or comma list of ICE URLs).
- **Push envs:** `PUSH_VAPID_PUBLIC_KEY`, `PUSH_VAPID_PRIVATE_KEY`, `PUSH_VAPID_SUBJECT` (web push), `PUSH_FCM_PROJECT_ID`, `PUSH_FCM_CLIENT_EMAIL`, `PUSH_FCM_PRIVATE_KEY` (Android FCM), and `PUSH_APNS_KEY_ID`, `PUSH_APNS_TEAM_ID`, `PUSH_APNS_BUNDLE_ID`, `PUSH_APNS_PRIVATE_KEY`, `PUSH_APNS_USE_SANDBOX` (iOS APNs).
- **Auth bootstrap envs:** `AUTH_BOOTSTRAP_PRIVATE_KEY` (ES256 JWK used by the API to sign offline bootstrap tokens), `VITE_AUTH_BOOTSTRAP_PUBLIC_KEY`/`AUTH_BOOTSTRAP_PUBLIC_KEY` (public JWK used by the client to verify offline bootstrap tokens).
- **SpacetimeAuth envs:** `SPACETIMEAUTH_AUTHORITY`, `SPACETIMEAUTH_CLIENT_ID`, `SPACETIMEAUTH_JWKS_URI`, `SPACETIMEAUTH_POST_LOGOUT_REDIRECT_URI` on the API/server side, plus `VITE_SPACETIMEAUTH_AUTHORITY`, `VITE_SPACETIMEAUTH_CLIENT_ID`, `VITE_SPACETIMEDB_URI`, and `VITE_SPACETIMEDB_MODULE` for the site bundle and direct SpacetimeDB clients.
- **Partytown envs:** `VITE_ENABLE_PARTYTOWN`/`ENABLE_PARTYTOWN` opt the site into rendering the Partytown loader snippet, and `VITE_PARTYTOWN_FORWARD`/`PARTYTOWN_FORWARD` accept a comma/newline list of globals to forward into the worker.
- **API base resolution:** Frontend resolves API origin via `API_BASE`/`VITE_API_BASE` (absolute URL or `/api` prefix). Default dev fallback is `http://127.0.0.1:4000`. Set the envs explicitly when front/back aren’t co-located.
- **Public base (IPFS/PWA):** `VITE_PUBLIC_BASE` controls the Vite `base` path (use `./` for IPFS/gateway hosting so assets resolve under the CID path).
- **P2P relay/signaling envs:** `VITE_P2P_RELAY_BASES` (HTTP mailbox relays), `VITE_P2P_NOSTR_RELAYS` (wss relays for discovery/prekeys), `VITE_P2P_WAKU_RELAYS` (Waku multiaddrs), and `VITE_P2P_CRDT_SIGNALING` (y-webrtc signaling list; if empty, falls back to `/yjs`).
- **Database bootstrap:** Compose dev/preview now ensures SpaceTimeDB JWT keys exist and publishes the Rust module into the running SpaceTimeDB instance before the API/site are treated as ready.
- **Compose profiles:** the `realtime` profile is owned by the `realtime` bundle and controls realtime behavior such as WebTransport and the embedded `/yjs` signaling endpoint; `full` enables it by default, while `core` leaves those features off.
- **Networking:** Caddy expects `prometheus.dev` to resolve to localhost. On WSL/non-macOS, set `DEV_WEB_UPSTREAM` if `host.docker.internal` is unsuitable.

## Compatibility and constraints

- **Runtimes:** Prefer Bun for scripts and package management. Avoid switching to npm/yarn. TypeScript target is modern (`typescript@6.0.0-dev`); Vite 8 beta + Qwik require up-to-date Node headers but the runtime is Bun.
- **Fragments:** Keep fragment payloads binary-compatible with `packages/core/src/fragment/binary.ts` and API encoders. Changes to fragment schemas must update both sides and related tests.
- **Early hints:** Fragment plans may include `earlyHints` for shell assets only (CSS, fonts, critical JS); never include fragment payloads or WebTransport URLs.
- **Caching:** Garnet cache keys for store items come from `buildStoreItemsCacheKey`; invalidation is coupled to realtime events. Preserve this coupling when modifying store logic.
- **Rate limits and payload limits:** Respect API constraints in `packages/platform-rs/src/app.rs` (prompt length, body size, WS quotas). Frontend UX should surface these limits rather than bypass them.
- **TLS/hosts:** Dev HTTPS assumes mkcert-style certs under `infra/caddy/certs` (shared with Caddy and WebTransport). Don’t check private keys into version control; reuse existing paths.
- **WebTransport TLS:** Chrome may require WebTransport developer mode for mkcert/local CAs (`chrome://flags/#enable-webtransport-developer-mode` or launch with `--enable-features=WebTransportDeveloperMode`; `chrome-devtools-mcp` supports `--acceptInsecureCerts`/`--chromeArg`).
- **Service worker policy:** Browser and PWA builds can generate/register the service worker; keep that behavior isolated from SSR and preview-only flows.

## Repo conventions and checks

- **Scripts:** Keep the root surface template-facing: `dev`, `dev:core`, `desktop:dev`, `desktop:run`, `desktop:build`, `desktop:build:canary`, `desktop:build:stable`, `build`, `build:core`, `preview`, `typecheck`, `typecheck:core`, `desktop:typecheck`, `test`, `test:core`, `template:init`, `template:sync`, `check:template`, `test:browser:full`, and `test:browser:core`. Internal helpers should stay app-local or `_internal:*`. API linting uses Oxlint configs in `packages/platform/.oxlintrc.json`.
- **Testing:** Root `bun run test` executes API tests; `bun run typecheck` covers site, Storybook config, and packages. Add targeted tests in `packages/platform/tests/`, `apps/site/src/**/*.test.tsx`, or `tests/browser/*.spec.ts`.
- **Template sync:** `bun run template:sync` regenerates `docs/template-reference.md`, the manifest, and env examples. `bun run check:template` verifies generated docs, branding placeholders, and untracked build outputs.
- **Native affordance fallbacks:** `apps/site/src/native/affordances.ts` and `apps/site/src/native/haptics.ts` must preserve browser/PWA UX without assuming platform plugins are available.
- **Preferences key allowlist:** Lightweight settings persisted via `apps/site/src/native/preferences.ts` are limited to `theme`, `locale`, `haptics-enabled`, `onboarding-complete`, and `last-tab`. Keep additional user/session/cache data in existing storage layers.
- **Formatting:** API files use Oxlint/formatter configs (`.oxlintrc.json`, `.oxfmtrc.json`). Frontend follows project styling in `packages/ui/src/global.css` and component patterns (Qwik components with `$` suffix).
- **Git hooks:** `.githooks` remains the hook source; wire it manually if you need local hooks.

## File map (quick pointers)

- **Site:** `apps/site/src/root.tsx` (app shell), `apps/site/.storybook/` (Storybook config), `apps/site/src/routes/` (pages/layout/head), `apps/site/src/features/fragments/` (FragmentShell), `apps/site/src/fragment/definitions/` (site fragment bundles).
- **Desktop:** `apps/desktop/src/bun/index.ts` (Electrobun main process), `apps/desktop/electrobun.config.ts` (desktop build/runtime config).
- **Stories:** `apps/site/src/**/*.stories.tsx` and `packages/ui/src/**/*.stories.tsx` feed the shared Storybook instance; prefer real component stories over generated onboarding samples.
- **Core:** `packages/core/src/fragment/` (types/codec/planner/service), `packages/core/src/app/` (client extras).
- **Template config:** `packages/template-config/src/index.ts` (bundle manifests, presets, and shared feature resolution).
- **Platform/API:** `packages/platform-rs/src/main.rs` (API/runtime entrypoint), `packages/platform-rs/src/config.rs` (resolved runtime config), `packages/platform-rs/src/fragments.rs` (fragment routes).
- **Features:** `packages/platform-rs/src/auth.rs`, `packages/platform-rs/src/store.rs`, `packages/platform-rs/src/chat.rs`, `packages/platform-rs/src/home_collab.rs`, `apps/site/src/features/lab/lab-route.tsx`.
- **Infra:** `docker-compose.yml` (service graph), `infra/caddy` (Caddyfile routing), `infra/spacetimedb/keys`, `scripts/*.ts` (compose helpers, preview/dev).
- **Runtime config:** `scripts/runtime-config.ts`.
- **WebTransport:** integrated into `packages/platform-rs`; no separate sidecar entrypoint.

## Contribution dos and don’ts

- **Do** keep frontend/API contracts in sync, especially fragment schemas and cache headers.
- **Do** prefer HTTPS + Caddy flow for testing view transitions and HMR in dev.
- **Do** document new env vars and update this file when site behavior or compatibility assumptions change.
- **Do** treat generated artifacts as build outputs:
  - `apps/site/public/fragments/`
  - `apps/site/src/fragment/fragment-css.generated.ts`
  - `infra/caddy/Caddyfile`
- **Don’t** replace Bun tooling with npm/yarn or add global installs when a Bun script exists.
- **Don’t** bypass rate limiting, cache invalidation hooks, or fragment sanitization paths.

When in doubt, mirror existing patterns in the same directory and keep fragment + API interfaces aligned. Update this document if you introduce new rules or workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Collin-Budrick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Collin-Budrick)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
