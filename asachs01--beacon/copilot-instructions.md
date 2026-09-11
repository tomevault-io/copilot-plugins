## beacon

> - React 19 + TypeScript + Vite SPA

# Beacon — Project Instructions

## Architecture

- React 19 + TypeScript + Vite SPA
- Home Assistant add-on with ingress support
- Capacitor for native iOS/Android builds

## Learnings - 2026-03-29

### Dual-Directory Structure
HA builds from `beacon/` subdirectory, NOT the repo root. After ANY source change, sync:
```bash
rsync -av --delete src/ beacon/src/
cp Dockerfile run.sh CHANGELOG.md beacon/
```
Both `config.yaml` and `beacon/config.yaml` must have matching versions.

**The root-level `Dockerfile` builds from repo-root context (`context: .` in
`.github/workflows/build-addon.yml`), not from `beacon/`.** So anything that
Dockerfile `COPY`s -- including `custom_intents/` and `custom_sentences/` --
must also exist at the repo root, mirrored from `beacon/`, or the Docker
build fails with `"failed to calculate checksum ... not found"`. This bit
us once already: those two dirs existed only under `beacon/` and the image
build silently never ran for months (see the CI/release-trigger bug below),
so nobody caught it until the pipeline was fixed and the build actually
executed. When adding root-level content Dockerfile depends on, sync it
the same way:
```bash
rsync -av --delete beacon/custom_intents/ custom_intents/
rsync -av --delete beacon/custom_sentences/ custom_sentences/
```

### HA Add-on Auth: Long-Lived Token in Config
SUPERVISOR_TOKEN only works container-side (http://supervisor/core). postMessage auth doesn't work in HA companion app WKWebView. The `ha_token` config option (schema: password) with a user-provided long-lived access token is the only reliable browser-side auth approach.

### HA Todo Items: Service Call with ?return_response
Don't read `entity.attributes.items` — it doesn't exist. Use:
```
POST /api/services/todo/get_items?return_response
Body: {"entity_id": "todo.xxx"}
Response: { service_response: { "todo.xxx": { items: [...] } } }
```
Filter out `unavailable` entities during discovery. Calling get_items on unavailable entities returns HTTP 500.

### HA Add-on Store Cache Busting
HA aggressively caches add-on repos. To force update visibility: create a git tag + GitHub release. If that fails, user must remove and re-add the repo URL. The `update_entity` service does NOT trigger a repo refresh.

### Semantic Release
`.releaserc.json` and `.github/workflows/release.yml` handle automated versioning. Uses conventional commits (`fix:` → patch, `feat:` → minor, `BREAKING CHANGE:` → major). Bumps both config.yaml files, syncs changelogs, creates GitHub releases.

---
> Source: [asachs01/beacon](https://github.com/asachs01/beacon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
