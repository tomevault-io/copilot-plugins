## elastic-stack-version

> >-


# Elastic Stack Version — Always Latest

## Current Version

**`9.4.0-SNAPSHOT`** (update this when a new version is released)

## Where It Applies

Every Docker image tag for Elastic components must use the current version:

- `docker.elastic.co/elasticsearch/elasticsearch:9.4.0-SNAPSHOT`
- `docker.elastic.co/kibana/kibana:9.4.0-SNAPSHOT`
- `docker.elastic.co/apm/apm-server:9.4.0-SNAPSHOT`
- `docker.elastic.co/beats/elastic-agent:9.4.0-SNAPSHOT`
- `docker.elastic.co/beats/filebeat:9.4.0-SNAPSHOT`
- `docker.elastic.co/beats/metricbeat:9.4.0-SNAPSHOT`

## Files to Check

When creating or modifying any of these files, ensure the version is current:

| File | What to check |
|------|--------------|
| `docker/docker-compose.yml` | All `image:` tags |
| `docker/docker-compose.lite.yml` | All `image:` tags |
| `.github/workflows/*.yml` | Service container `image:` tags |
| `examples/**/docker-compose.yml` | All `image:` tags |
| `scripts/seed-test-data.sh` | Beat index name versions (e.g. `filebeat-9.4.0-SNAPSHOT-*`) |

## npm Client Package

The `@elastic/elasticsearch` npm client must also track the current major version:

**`^9.0.0`** (currently resolves to `9.3.4`)

| File | What to check |
|------|--------------|
| `packages/*/package.json` | `@elastic/elasticsearch` dependency version |
| `examples/*/package.json` | `@elastic/elasticsearch` dependency version |

After updating, run `npm install` in the repo root to refresh the lockfile.

## What NOT to Change

- **Test fixture strings** in unit tests (e.g. `version: '8.17.0'` in mock data) —
  these are test data, not infrastructure.
- **Documentation examples** that reference specific historical versions for context.

## Auto-Fix

When you see an outdated Elastic stack Docker image version:
1. Replace it with the current version
2. Check ALL files in the list above — version drift across files is common
3. If updating docker-compose, also run `docker compose down -v` to clear stale volumes
   (older ES versions can't read data from newer ones and vice versa)

## Pre-Upgrade Validation — MANDATORY

Before changing the version, complete this checklist. Do NOT skip steps.

### 1. Image Availability
- SNAPSHOT images (`X.Y.Z-SNAPSHOT`) are built from `main` and may not be available for
  all components (APM Server, Beats, Elastic Agent). Verify before assuming.
- Check `docker.elastic.co` for the image tag. If it doesn't exist, consider using the
  latest released version instead or note the risk.
- ARM64 (Apple Silicon) images may lag behind x86_64 for SNAPSHOT builds.

### 2. Breaking Changes
- Read the [Elastic release notes](https://www.elastic.co/guide/en/elasticsearch/reference/current/release-notes.html)
  for any breaking changes between the current and target version.
- Check if index compatibility mode changed (ES won't read indices from incompatible versions).
- Check if API endpoints were removed, renamed, or had default behavior changes.
- Check if security defaults changed (e.g., security enabled by default in 8.x+).

### 3. Client Compatibility
- The `@elastic/elasticsearch` JS client must support the target server version.
  Client 9.x supports ES 9.x; client 8.x supports ES 8.x. Cross-major is NOT guaranteed.
- After upgrading, verify the client resolves to a compatible version in the lockfile.

### 4. Test Data Compatibility
- Beat index names include the version (e.g., `filebeat-9.4.0-SNAPSHOT-2024.01.15`).
  Seed scripts must use the new version.
- Index mappings may change between versions. Verify seed data schemas are still valid.
- Data stream naming conventions may change.

### 5. CI Pipeline Impact
- CI service containers must use the same version as docker-compose.
- SNAPSHOT images may not be cached on CI runners — builds may be slower.
- Verify the CI runner has access to `docker.elastic.co` for the target image.

### 6. Cross-Repo Sync
After upgrading in one repo, check these sibling repos for version drift:
- `~/Projects/elastic-cursor-plugin` — Docker, CI, examples, seed scripts
- `~/Projects/cursor-plugin-evals` — Docker, CI, showcase examples
- Any other repo with `docker.elastic.co` image references

### 7. Post-Upgrade Verification
After all files are updated:
1. Run `npm install` and verify lockfile changes
2. Run `tsc --noEmit` to catch type-level breaking changes
3. Run `npm test` to catch runtime breaking changes
4. If Docker is available, run `docker compose down -v && docker compose up -d` and verify
   the cluster comes up healthy
5. Verify the seed script runs without errors against the new version

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
