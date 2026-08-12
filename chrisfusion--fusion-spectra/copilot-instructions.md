## fusion-spectra

> Vue 3 + Quasar 2 + Vite micro-frontend shell (Module Federation host).

# fusion-spectra

Vue 3 + Quasar 2 + Vite micro-frontend shell (Module Federation host).
Dev server: `npm run dev` → http://dev.fusion.local:5174
Requires `127.0.0.1 dev.fusion.local` in `/etc/hosts` (localhost breaks SameSite=Lax cookie sharing with bff.fusion.local)
Vite config: `server.host:'0.0.0.0'`, `server.port:5174`, `server.allowedHosts:['dev.fusion.local']`
Type check: `npm run typecheck`

## CHANGELOG maintenance
- `CHANGELOG.md` is the project changelog — update it for every feature and bugfix going forward
- Format: add entries under `## [Unreleased]` with today's date as a comment; use `### Added`, `### Changed`, `### Fixed` subsections; one line per item
- When a deployment bumps the image tag (values-dev.yaml), also promote `[Unreleased]` to a versioned section (e.g. `## [0.9.1] — YYYY-MM-DD`) matching the new tag
- Use `date +%Y-%m-%d` in Bash to get today's date when writing changelog entries

## README maintenance
- `README.md` feature-status claims (Live/Placeholder) drift from reality easily — verify against `src/pages/<context>/` file listing and `src/data/navigation.ts` before trusting or updating them
- `INSTALL.md` does not exist (never committed, despite historical references) — don't link to it; point setup/deployment docs at `CLAUDE.md` and `ARCHITECTURE.md` instead

## Stack
- Vue 3, Quasar 2, Pinia, Vue Router 4, Vite 5
- Icons: `@quasar/extras` mdi-v7 (use `mdi-*` names)
- Fonts: DM Sans (UI), JetBrains Mono (data/mono) — loaded via Google Fonts in index.html
- CSS custom properties in `src/css/app.scss` (all `--fs-*`)
- Quasar theme vars in `src/css/quasar-variables.scss`

## Layout architecture
- `src/layouts/MainLayout.vue` — shell: topbar + activity rail + sidebar + canvas
- Activity rail (`src/components/ActivityRail.vue`) — three-zone model: regular (top) → separator (flex:1) → util (bottomUtil) → admin (adminOnly); admin section gated by `isAdmin`, util section always visible
- Sidebar (`src/components/AppSidebar.vue`) — IDE-style tree; 3-level when `NavGroup.section` is set (renders a small uppercase section header when label changes across sibling groups); `NavLeaf.tooltip` renders a `q-tooltip` on hover (400ms delay, anchors right)
- Canvas panels use `src/components/CanvasPanel.vue`
- Context/nav data: `src/data/navigation.ts` — single source of truth

## CanvasPanel component
Props: `title`, `icon`, `wide` (span 2 cols), `loading` (spinner overlay), `error` (error state + retry).
Emits: `refresh`. Slots: default body, `actions` (header right area).
No footer slot — add pagination below the table inside the default slot.

## Contexts (activity rail order)
1. Data → `/data`
2. Weave → `/pipelines` (label "Weave", id `pipelines`) — groups use `section` to form two topics: **Runs** (Monitoring + Control sub-groups) and **Blueprints** (Run Blueprints + Step Blueprints); all "templates" renamed to "blueprints" in labels
3. Monitoring → `/monitoring`
4. Forge → `/forge` — async Python venv builder (fusion-forge backend)
5. Fusion Index → `/fusion-index` — live registry UI backed by fusion-index API
6. Admin → `/admin` (admin-only, amber accent, bottom of rail)

## Auth
- BFF owns all OIDC — frontend knows nothing about Keycloak or tokens
- Auth store (`src/stores/auth.ts`): `init()` calls `GET /bff/userinfo` with `credentials:'include'`; 401 → `window.location.href = bffUrl + '/bff/login'`
- `UserInfo` shape: `{ sub, email, name, roles: string[], permissions: string[], resource_permissions: ResourcePermission[] }` — populated from BFF session
- `ResourcePermission` shape: `{ permission: string, resource_type: string, resource_id: string }` — resource-scoped grants
- Router guard in `src/router/index.ts` calls `auth.init()` on every navigation; routes with `meta.adminOnly: true` redirect non-admins to `/data`
- BFF URL from `src/config/runtime.ts` → `window.FUSION_CONFIG.bffUrl` → `VITE_BFF_URL` → `http://bff.fusion.local`
- Runtime config file: `public/config.js` (overridden by ConfigMap mount in K8s)

## RBAC (permissions)
- `src/composables/usePermission.ts` — call `usePermission()` in any component that needs access control
  - `can(permission: string, resourceId?: number | string)` — true if user has global permission OR a resource-scoped grant for that ID
  - `hasRole(role: string)` — true if `auth.user.roles` contains the role
  - `isAdmin` — computed: `hasRole('admin')`
- Gate UI elements with `v-if="can('index:artifacts:delete')"` etc., NOT with role checks — roles are too coarse for UI gates
- Resource-scoped gating: `v-if="can('index:artifacts:delete', artifact.id)"` — true only if user has global perm OR a specific grant for that resource
- Admin icon in ActivityRail is hidden via `v-if="isAdmin"` — no admin entry renders for non-admin users
- Admin routes (`/admin/*`) have `meta.adminOnly: true`; the router guard redirects to `/data` if user lacks `admin` role
- Permission strings mirror the BFF `rbac.yaml` (e.g. `index:artifacts:read`, `forge:builds:create`, `admin:roles:manage`)

## API clients
- `src/api/bffClient.ts` — base fetch with `credentials:'include'`; 401 auto-redirects to BFF login
  - FormData detection: skips `Content-Type: application/json` when `body instanceof FormData` (multipart uploads)
  - `bffDelete` returns `Promise<void>` and discards the response body — for DELETE endpoints that return JSON (e.g. bulk-delete result), use `bffFetch(path, { method: 'DELETE' })` then `.json()` directly
  - Non-2xx handling only reads a flat `body.error` string — a structured error body (e.g. `{valid, errors: [{line, message}]}`) collapses to `res.statusText`, losing detail. Endpoints needing field-level errors should return 200 with a `valid`-style flag to check directly (e.g. `/batchtriggers/validate`), not rely on catching a 4xx.

## Shared utilities & components
- `src/utils/format.ts` — `formatSize(bytes)`: human-readable file size (B / KB / MB / GB)
- `src/components/TagChipInput.vue` — v-model `string[]` chip input; Enter/comma adds, Backspace removes last, × removes specific; validation `/^[a-zA-Z0-9-]+$/` max 64 chars; trailing commas stripped
- `src/components/JsonEditor.vue` — CodeMirror 6 JSON editor; emits `valid` (false on non-empty invalid JSON); `{ } Format` button pretty-prints; `defineExpose({ format })` for programmatic use; theme via `--fs-*` CSS vars
- `src/components/CronPicker.vue` — v-model on a cron-expression string; presets dropdown (every 5/15/30 min, hourly, daily, weekly, monthly) + "Custom (advanced)" raw-expression fallback + live human-readable summary; defaults to daily 09:00 rather than a blank field

## Themes
- `src/stores/theme.ts` — 5 themes: lumen (default), azure, carbon, matrix, synthwave; persisted to localStorage; `midnight`/`light` are gone — a `Set` guard coerces stale localStorage values to `lumen`
- Applies `data-theme` on `<html>` + calls `Quasar.Dark.set()` — CSS vars alone don't affect Quasar portals (menus, tooltips)
- CSS variable overrides per theme in `src/css/app.scss` under `[data-theme="<name>"]` blocks

## Adding a new page / feature
1. Add leaf to `src/data/navigation.ts` under the correct group
2. Add route to `src/router/index.ts` — literal paths (`/create`) before dynamic (`/:id`)
3. Create page component under `src/pages/` (or `src/pages/<context>/`)
Both navigation.ts and router must be updated together — neither works without the other.

## Multi-step wizard pattern
Used in `ArtifactCreatePage`, `ArtifactVersionCreatePage`, and all weave wizards:
- `step` ref (number), `v-if="step === N"` per section inside one CanvasPanel
- Validate on Next click; only advance if valid
- Track partially-created resources in a ref (e.g. `createdId`) — prevents duplicate creation on retry and enables orphan recovery UI
- For split-panel step 2 (builder + live preview): use CSS grid `1fr NNNpx`; preview col `position:sticky;top:16px`
- Stable v-for keys: add a `uid: number` field to row interfaces; use `++_uid` counter; never use array index as key on deletable lists

## Quasar + Vite gotchas
- `sass-embedded` must be in `devDependencies` (not `dependencies`)
- `sassVariables` path in `@quasar/vite-plugin` must be absolute (`resolve(__dirname, ...)`)
- `build.target: 'esnext'` required when Module Federation is added
- Import mdi css before quasar css in `main.ts`
- Do NOT set `config: { dark: true }` in main.ts — let the theme store call `Dark.set()` instead; otherwise light theme still renders dark Quasar components
- API fields like `types[]` and `tags[]` may be absent from responses even when typed — always guard with `?? []`
- Vue 3: `ref` inside `v-for` resolves to an **array** — declare as `ref<El[]>([])` and access `[0]`; a `ref<El|null>(null)` silently becomes an array and `.focus()` fails
- Vue 3: `Set.add()` / `Set.delete()` are NOT reactive — replace the whole ref: `s.value = new Set([...s.value, x])`
- `--fs-bg-panel` is NOT defined in `app.scss` — use `--fs-bg-elevated` or `--fs-bg-surface` for solid backgrounds; `--fs-bg-panel` resolves to transparent
- `q-dialog` and `q-tooltip` render as portals outside component DOM — CSS overrides must be in an unscoped `<style>` block (not `<style scoped>`)
- Dialog-scoped polling: use `watch(dialogOpenRef, open => { if (!open) stopPolling() })` to tie a timer's lifecycle to a `q-dialog`; cleaner than hooking every close path individually
- `@codemirror/lint` is a separate npm package — install explicitly; `lintGutter` lives there, not in `@codemirror/language`
- Scoped styles don't cross component boundaries: shared components (`TagChipInput.vue`, `JsonEditor.vue`, `CronPicker.vue`) must define their own scoped CSS classes using the global `--fs-*` vars — they can't reuse a parent page's `.fs-input`/`.fs-btn`-style classes, which are scoped to that page's own `<style>` block

## Screenshots
`screenshots/` — UI screenshots named `YYYY-MM-DD_<description>.png`
- Test/debug screenshots **must** use the prefix `test_` (e.g. `test_2026-05-11_wizard-step2.png`) — these are git-ignored

## UI testing (Playwright)
- Test against minikube at `http://spectra.fusion.local`, not the dev server
- Use `browser_snapshot` (not screenshot) to get element `ref` values for clicks/fills
- Use `browser_take_screenshot` with `fullPage: true` to save to `screenshots/`
- After pod restart, browser may serve cached JS — navigate to `/#/` first; if the canvas is completely blank (`<!---->`) across all routes, the Playwright browser has stale JS and needs `location.reload(true)` — navigating to `/#/` alone is not sufficient in that case
- Quasar menus/dropdowns are portals — they don't appear in `browser_snapshot`; use `browser_evaluate` to read/click items inside `.q-menu`
- `q-dialog` portals render elements in BOTH the component DOM and `#q-portal--dialog--N` — `button[title="..."]` selectors trigger Playwright strict-mode violations; use class-based selectors (e.g. `.log-dialog__controls button:first-of-type`) instead
- Access Pinia stores in evaluate: `document.querySelector('#app').__vue_app__.config.globalProperties.$pinia._s.get('storeName')`; theme store action is `.set(themeName)`
- Playwright browser has its own separate session and cache from the user's browser — stale `config.js` there doesn't mean the user has the same problem
- To call BFF APIs with auth during testing: `browser_evaluate` with `fetch('http://bff.fusion.local/api/...', { credentials: 'include' })` — uses the browser's session cookies

## Activity rail — utility zone
- `Context.bottomUtil?: boolean` — renders between separator and admin; always visible (no `isAdmin` guard); use for standalone nav buttons with no sidebar
- Set `groups: []` on a `bottomUtil` context; `MainLayout` detects empty groups and navigates directly without opening the sidebar
- To add a new utility button: add entry to `navigation.ts` with `bottomUtil: true, groups: []`, add route to `router/index.ts`

## Deployment (fusion-spectra)
- Dockerfile: 3-stage (deps → build → nginx:alpine); `NPM_REGISTRY` build arg for private registry
- `nginx.conf`: `location /` (SPA fallback) has `no-store, no-cache` — critical so browsers always fetch fresh `index.html` after redeploy; **`location /` must carry the no-cache header** — putting it only on the regex location leaves the root path cacheable and causes blank pages after redeploy
- nginx runs as non-root (`USER nginx`, uid 101) on **port 8080**; `fsGroup: 101` makes emptyDir mounts writable without an initContainer
- Always use semver image tags (never `latest`/`local`): `eval $(minikube docker-env) && docker build -t fusion-spectra:X.Y.Z .`
- Docker build MUST run inside minikube's daemon (`eval $(minikube docker-env)` first) — otherwise pod gets `ErrImageNeverPull`
- After building, update `image.tag` in `values-dev.yaml` and run `helm upgrade`; tag change triggers pod replacement automatically
- Helm field manager conflict: if `kubectl set image` was used, bypass with `kubectl set image deployment/fusion-spectra frontend=fusion-spectra:X.Y.Z -n fusion`
- Stale probe ports: if nginx moved to 8080 but probes still hit 80, pods crash-loop — patch: `kubectl patch deployment fusion-spectra -n fusion --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/livenessProbe/httpGet/port","value":8080},...]'`
- Stale JS chunks after redeploy: `router.onError` + `unhandledrejection` + `vite:preloadError` in `src/router/index.ts` auto-reload on chunk-not-found; guard is **timestamp-based** (8s cooldown in `__chunk_reload_ts__`) — do NOT use a boolean flag, Ctrl+Shift+R does not clear `sessionStorage` so a boolean guard permanently blocks recovery
- "Clean reinstall on minikube" = `eval $(minikube docker-env) && docker build -t fusion-spectra:X.Y.Z . && kubectl set image deployment/fusion-spectra frontend=fusion-spectra:X.Y.Z -n fusion`

## Runtime config gotchas
- Adding a new `FUSION_CONFIG` field requires updating BOTH `deployment/values.yaml` AND `deployment/templates/configmap.yaml` — missing either silently drops the field in K8s
- After a ConfigMap change + pod restart, browsers need a hard refresh (`Ctrl+Shift+R`) to pick up the new
  `config.js` — it's served with `Cache-Control: public, immutable, max-age=31536000` (matched by nginx's
  static-asset rule, unlike `index.html`'s no-cache rule), so a normal reload isn't enough. In Playwright,
  a hard reload may still serve the cached copy from the shared browser-context disk cache — use a CDP
  session (`page.context().newCDPSession(page)` → `Network.clearBrowserCache`) or a fresh browser context.
- Direct ConfigMap patch when helm upgrade has a field manager conflict: `kubectl create configmap <name> --from-literal=config.js='...' --dry-run=client -o yaml | kubectl apply -f - && kubectl rollout restart deployment/<name> -n fusion`

## fusion-bff deployment (minikube)
- Build: `cd /path/to/fusion-bff && eval $(minikube docker-env) && docker build -t fusion-bff:X.Y.Z .`
- Deploy: `kubectl set image deployment/fusion-bff fusion-bff=fusion-bff:X.Y.Z -n fusion && kubectl rollout status deployment/fusion-bff -n fusion`
- Container name in the deployment is `fusion-bff` (not `bff`) — required for `kubectl set image`
- BFF session store is **in-memory** — every pod restart wipes all sessions; Playwright browser (and users) must re-login after any BFF redeploy
- Adding a new BFF permission requires BOTH: (1) add to `role_permissions` for each relevant role in `rbac.yaml`, AND (2) add a `route_permissions` rule before the catch-all; missing either silently blocks the request
- `rbac.yaml` is mounted from the `fusion-bff-rbac` ConfigMap, not baked into the image — adding permissions to the source file does NOT update the running cluster; patch with: `kubectl create configmap fusion-bff-rbac --from-file=rbac.yaml=<file> -n fusion --dry-run=client -o yaml | kubectl apply -f - && kubectl rollout restart deployment/fusion-bff -n fusion`
- The deployed ConfigMap can be stale vs the source — check with `kubectl get configmap fusion-bff-rbac -n fusion -o jsonpath='{.data.rbac\.yaml}'` before assuming permissions are live
- `helm upgrade fusion-bff` fails locally with "config.oidcIssuerUrl is required" — use `kubectl set image` + manual rbac ConfigMap patch instead

## Local DNS for *.fusion.local (minikube)
- No more manual `/etc/hosts` entries for minikube ingress hosts (`spectra.`, `bff.`, `index.`, `gitea.`,
  `keycloak.fusion.local`, and any future ones) — a dedicated `dnsmasq` instance on the host resolves the
  whole `*.fusion.local` wildcard to the current minikube IP
- `dev.fusion.local` is the one exception — stays in `/etc/hosts` as `127.0.0.1` (Vite dev server, unrelated
  to minikube); `files` beats `dns` in `/etc/nsswitch.conf`, so don't add a `192.168.49.2` hosts entry for
  any other `*.fusion.local` name or it'll silently shadow the DNS answer
- Components: `/etc/dnsmasq-fusion.conf` (`address=/fusion.local/<minikube-ip>`, listens on
  `127.0.0.1:5353` — a non-standard port so it doesn't collide with systemd-resolved's stub on
  `127.0.0.53/54:53`), `dnsmasq-fusion.service`, and `/etc/systemd/resolved.conf.d/fusion-local.conf`
  (`Domains=~fusion.local` routes only that domain to the dnsmasq instance — everything else still uses the
  normal upstream)
- Self-healing: `fusion-dns-sync.timer` (every 60s) re-reads `minikube ip` and rewrites
  `/etc/dnsmasq-fusion.conf` + restarts `dnsmasq-fusion.service` if it drifted (e.g. after
  `minikube delete && start`) — no manual fixup needed
- Check it's working: `resolvectl query spectra.fusion.local` should return the minikube IP; if not, check
  `systemctl status dnsmasq-fusion fusion-dns-sync.timer`

## Help system
- `src/components/HelpDrawer.vue` — slide-over drawer; "This page" tab loads route-scoped articles + videos; "Browse all" tab has search + service/type filters; article body rendered from Markdown via `marked`
- `src/composables/useHelpDrawer.ts` — `provideHelpDrawer()` called in `MainLayout`; `useHelpDrawer()` injects in any child; returns `{ open: Ref<boolean>, toggle, close }`
- `src/pages/HelpPage.vue` — full-page help browser at `/help`; `bottomUtil` nav entry with `groups: []`
- **Page Guide** — global toggle in `AppTopBar.vue` (`mdi-compass-outline`); uses `useHelpDrawer()` inject; this is the primary entry point — do NOT add per-panel `help` props to CanvasPanel instances
- Activity rail help entry uses `mdi-book-open-outline` (navigates to `/help`); topbar uses `mdi-compass-outline` (opens drawer) — keep these distinct
- `routes:` in help article frontmatter must be **exact static paths** — parameterised paths will NOT match; use the list page path instead
- Help articles live in `help/<service>/<type>/<slug>.md`; service IDs: `forge`, `weave`, `index`, `admin`, `data`, `monitoring`
- To update the fusion-content repos Secret: edit `/tmp/repos.yaml`, then `kubectl -n fusion create secret generic fusion-content-repos --from-file=repos.yaml=/tmp/repos.yaml --dry-run=client -o yaml | kubectl apply -f -` + `kubectl -n fusion rollout restart deployment/fusion-content-server`

## fusion-content API
- BFF proxies `/api/content/*` → fusion-content service; permission `content:changelog:read`
- `GET /api/content/api/v1/changelog?page=1&pageSize=20` → `{ data: DateGroup[], pagination }`; `DateGroup`: `{ date, projects: ProjectEntry[] }`; `ProjectEntry`: `{ project, version, changes: { added?, changed?, fixed?, removed? } }`
- `GET /api/content/api/v1/help` params: `service`, `type`, `tag`, `route`, `q`, `page`, `pageSize` → `{ data: HelpArticle[], pagination }`; `HelpArticle`: `{ service, type: DiátaxisType, slug, title, tags, routes, summary }`
- `GET /api/content/api/v1/videos` params: `service`, `pageSize` → `{ data: VideoItem[], pagination }`; `VideoItem`: `{ service, slug, title, summary, thumbnailUrl, videoUrl, tags }`

---
> Source: [chrisfusion/fusion-spectra](https://github.com/chrisfusion/fusion-spectra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
