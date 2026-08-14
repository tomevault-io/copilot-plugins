## obs-packaging

> Validates local project configuration without connecting to OBS.

# Percona OBS Packaging - AI Coding Instructions

## Project Purpose

This repo contains RPM and Debian **packaging metadata** for building Percona software packages via a self-hosted [OpenSUSE Build Service (OBS)](https://build.opensuse.org/) instance. It does **not** contain upstream source code — only packaging files. Sources are fetched at build time by OBS services declared in `obs/_service`.

- `osc` — the OBS CLI client (Python library, also used programmatically)
- `percona-obs` — the management script in this repo (see `requirements.txt`)
- `root/` — all packaging content lives here, mirroring the OBS project/package hierarchy

## Repository Layout

```
root/
├── project.yaml                    # OBS project config for the root project
├── common/                         # shared packages used across products
│   └── deps/
│       ├── build/                  # build-time deps (Go toolchain, OBS source services)
│       │   └── <package>/
│       │       └── obs/_aggregate  # aggregates from an external OBS project
│       └── runtime/                # runtime deps shared across products
│           └── <package>/          # e.g. percona-telemetry-agent
│               ├── debian/
│               ├── rpm/
│               └── obs/_service
└── <product>/                      # e.g. ppg/
    ├── releases/                   # release pointer files (see root/README.md)
    │   └── <name>/release.yaml
    ├── staging/                    # full package set, tag builds, QA/release candidate
    │   └── <major-version>/        # e.g. 17/
    │       ├── project.yaml        # OBS project config for this subproject
    │       ├── <package>/          # source packages
    │       │   ├── debian/         # Debian packaging (control, rules, changelog, …)
    │       │   ├── rpm/            # RPM packaging (*.spec, patches, service files)
    │       │   ├── package.yaml    # optional OBS package config (title, description)
    │       │   └── obs/
    │       │       ├── _service    # OBS build service config
    │       │       ├── _aggregate  # aggregates binaries from another OBS project
    │       │       └── _multibuild # multi-flavor builds (PostgreSQL extensions only)
    │       └── <another-package>/
    │           └── ...
    └── devel/                      # manually curated dev-branch subset (see below)
        └── <major-version>/
            └── <package>/          # Class A (full copy) or Class B (obs/_link only)
```

A directory is treated as a **package** if it contains an `obs/` subdirectory or a `package.yaml` file. Everything else is treated as a **project** (subproject grouping).

### `devel/<major-version>/` packages: Class A vs Class B

`devel/<V>/` holds a manually curated subset of `staging/<V>/`, built from development
branches so day-to-day branch work can be built and tested without disturbing staging.
Membership is manual — adding a package there also requires adding its direct dependents,
since an omitted dependent would otherwise link against staging binaries. A **Class A**
devel package is a full copy of the staging package (own `rpm/`, `debian/`, `obs/`) with
`obs/_service` retargeted from a release tag to a development branch, editable independently
of staging. A **Class B** devel package contains only an `obs/_link` pointing at the staging
package (e.g. `<link project="${OBS_ROOTPRJ}:ppg:staging:18" package="…"/>`), rebuilt in the
devel context so it links against devel binaries. See `root/README.md` for full detail.

## Two Package Archetypes

### 1. Standalone service (e.g., `common/deps/runtime/percona-telemetry-agent/`)
- Single static package name (no version placeholder)
- `obs/_service` fetches: packaging (debian + rpm subdirs) + upstream source + `go_modules` (manual)
- `debian/rules` extracts version from `.obsinfo` file at build time
- RPM `Release: 1%{?dist}`

### 2. PostgreSQL extension (e.g., `ppg/staging/17/percona-pg-telemetry/`)
- Uses `@BUILD_FLAVOR@` placeholder throughout (replaced by PG major version at build time)
- `obs/_multibuild` lists PG versions to build for: `<flavor>17</flavor>`
- `debian/pgversions` specifies min PG version (e.g., `9.3+`)
- RPM spec defines `%define pg_version @BUILD_FLAVOR@%{nil}` and uses `%{pgrel}` in `Name:`
- Built with PGXS: `USE_PGXS=1 make`

## Critical Conventions

**`obs/_service` structure** (all packages follow this pattern):
1. First `obs_scm` service: fetch `debian/` subdir from this repo — use `<param name="subdir">${DEBIAN_PACKAGE_DIRECTORY}</param>`
2. Second `obs_scm` service: fetch `rpm/` subdir from this repo — use `<param name="subdir">${RPM_PACKAGE_DIRECTORY}</param>`
3. Third `obs_scm` service: fetch upstream source from its canonical repo
4. Buildtime services: `tar`, `recompress` (gz), `set_version`
5. `go_modules` (manual mode) — only for Go projects (telemetry-agent, etcd)

**`debian/debian.dsc`** must list all tarballs in `Debtransform-Files-Tar`:
```
Debtransform-Files-Tar: debian.tar.gz vendor.tar.gz rpm.tar.gz
```

**Maintainer** (use consistently):
- `Percona Development Team <info@percona.com>` (Debian)
- `Percona LLC` (RPM)

**Epoch: 1** is set on PostgreSQL-related packages to allow version management.

## Project Configuration (project.yaml)

Each project directory may contain a `project.yaml` that defines its OBS project metadata.

```yaml
name:                          # optional — overrides the OBS project name (empty = use derived name)
title: My Project Title
description: "Human-readable description."
repositories:
  - name: RockyLinux_9         # OBS repository name
    paths:
      - project: openSUSE.org:RockyLinux:9   # upstream OBS project providing the build environment
        repository: standard
      - subproject: builddep   # relative reference: resolves to <rootprj>:builddep
        repository: RockyLinux_9
    archs: [x86_64]
project-config: |              # raw OBS project config string
  %if "%_repository" == "RockyLinux_9"
  ExpandFlags: module:llvm-toolset-rhel9
  %endif
```

- `name` — absent or empty means the OBS project name is derived from the directory path relative to `root/` joined with `--rootprj` using colons (e.g. `home:Admin:ppg:staging:17`). Set it explicitly only when the OBS project name must differ from the directory path.
- `repositories[].paths` — list of path entries providing the base build environment. Each entry uses either `project:` (absolute OBS project name) or `subproject:` (resolved as `<rootprj>:<subproject>`) plus `repository:`.
- `project-config` — passed verbatim to the OBS project config API; used for RPM macros, module expansion flags, etc.
- `title` and `description` are informational only and never inherited by child projects.

### Config inheritance

`repositories` and `project-config` are **inherited** from ancestor `project.yaml` files when absent or empty in a project's own file. The nearest ancestor that defines the field wins. `title`, `description`, and `name` are never inherited.

This means:
- The root `project.yaml` acts as the default config for all subprojects.
- A subproject only needs its own `project.yaml` if it requires a different build environment.
- An empty or missing `project.yaml` in a subdirectory is valid — it will fully inherit from its parent.

### Dynamically generated repository paths

When `percona-obs` pushes project metadata to OBS, it automatically injects one `<path>` entry per ancestor OBS project into every repository of every non-root subproject. This is done by `build_project_meta()` in `percona-obs`, using the `_ancestor_projects()` helper.

Ancestor paths are injected closest-first (immediate parent before grandparent), followed by the upstream path from `project.yaml`. This gives every subproject **direct** visibility into packages built in all ancestor projects, without relying on OBS transitive resolution.

For example, the `home:Admin:ppg:staging:17` project gets this generated for each repository:
```xml
<repository name="RockyLinux_9">
  <path project="home:Admin:ppg:staging" repository="RockyLinux_9"/>   <!-- auto-injected: immediate parent -->
  <path project="home:Admin:ppg" repository="RockyLinux_9"/>           <!-- auto-injected: grandparent -->
  <path project="home:Admin" repository="RockyLinux_9"/>               <!-- auto-injected: great-grandparent (rootprj) -->
  <path project="openSUSE.org:RockyLinux:9" repository="standard"/>   <!-- from project.yaml -->
  <arch>x86_64</arch>
</repository>
```

The root project (matching `--rootprj`) never gets ancestor paths injected. Only non-root subprojects are affected.

## Package Configuration (package.yaml)

Each package directory may contain a `package.yaml` with OBS package metadata:

```yaml
title: My Package Title
description: "Human-readable description."
```

These fields map directly to the OBS package `<title>` and `<description>` XML elements.

## `percona-obs` CLI

`percona-obs` is the management script for syncing local YAML configuration and packaging files to an OBS instance.

**Global options:**
```sh
# Using explicit flags (always works):
percona-obs -A <url> -R <rootprj> [--verbose] <command> ...

# Using a named profile (recommended for day-to-day use):
percona-obs -P <name> [--verbose] <command> ...

#   -A / --apiurl    OBS API URL (e.g. http://my-obs.local:8000)
#   -R / --rootprj   OBS root project (e.g. home:Admin)
#   -P / --profile   Load apiurl, rootprj, and env from .profile/<name>.yaml
#   -e KEY:VALUE     Define or override an env variable (repeatable; VALUE may be empty)
#   --verbose        Print debug-level log messages (API calls, unchanged items)
```
`-R` / `--rootprj` is always required — either directly or via a profile.  Explicit `-A` / `-R` / `-e` flags override the corresponding profile values when both are given.

OBS credentials are read from `~/.config/osc/oscrc` (created by `osc`'s first-run wizard).

### Connection profiles

Profiles store per-environment OBS connection settings in `.profile/<name>.yaml` (git-ignored). Create one file per environment; use `-P <name>` to activate it.

**File format** (`.profile/<name>.yaml`):
```yaml
apiurl: http://192.168.1.103:3000   # OBS API URL
rootprj: home:Admin:percona         # OBS root project
env:                                 # optional: variables for ${VAR} substitution
  - name: REMOTE_OBS_ORG_INTERCONNECT
    value: 'openSUSE.org:'           # values containing colons must be quoted
```

**Example** — create a `dev` profile and use it:
```sh
./percona-obs -A http://192.168.1.103:3000 -R home:Admin:percona \
  -e REMOTE_OBS_ORG_INTERCONNECT:'openSUSE.org:' \
  profile create dev

./percona-obs -P dev sync ppg:staging:17 etcd --dry-run
```

To add or update an env variable in an existing profile, use `-P` (to load the current state) plus `-e`:
```sh
./percona-obs -P dev -e ANOTHER_VAR:value profile create dev
```

If the named profile file does not exist, `percona-obs` exits with an error listing the profiles that are available in `.profile/`.

### Env variable substitution

`${VAR}` tokens in the following files under `root/` are substituted with values from the active profile's `env` section (or `-e` flags) before the content is used or uploaded to OBS:

- `project.yaml` and `package.yaml` — project/package metadata
- `obs/_service`, `obs/_aggregate`, `obs/_link` — OBS source files

This lets a single source tree target different OBS environments. For example, a local OBS instance interconnected to `build.opensuse.org` needs an `openSUSE.org:` prefix on external project references, while the public OBS does not:

```yaml
# root/project.yaml
repositories:
  - name: RockyLinux_9
    paths:
      - project: ${REMOTE_OBS_ORG_INTERCONNECT}RockyLinux:9
        repository: standard
```

```yaml
# .profile/dev.yaml  (local OBS with interconnect)
env:
  - name: REMOTE_OBS_ORG_INTERCONNECT
    value: 'openSUSE.org:'

# .profile/prod.yaml  (public build.opensuse.org — no prefix needed)
env:
  - name: REMOTE_OBS_ORG_INTERCONNECT
    value: ''
```

Use `project verify -P <profile>` to validate that all `${VAR}` tokens in the tree are defined in the given profile.

### Output format

`percona-obs` prints one line per resource, always — including unchanged ones. Each line has a two-character prefix, color-coded when stdout is a TTY (set `NO_COLOR=1` to disable):

| Prefix | Color | Meaning |
|---|---|---|
| `  + ` | green | Resource created on OBS (did not exist before) |
| `  ~ ` | yellow | Resource updated on OBS (existed, content changed) |
| `  = ` | dim | Resource unchanged (OBS already matches desired state) |
| `  - ` | red | Resource deleted from OBS (orphan cleanup or `sync delete`) |
| `  @ ` | cyan | Package aggregated from a branch source (`--branch-from`) |
| `  ! ` | yellow | Uncertain — OBS-only file skipped because services were not run (dry-run only) |
| `  > ` | cyan | Action taken: local service run or OBS service triggered |
| `  ✔ ` | bold green | Command completed successfully |
| `  · ` | dim | Debug message (only shown with `--verbose`) |

In dry-run mode the same `+`/`~`/`=`/`-`/`!` symbols are used — the `(dry run)` note on the final `✔` line indicates nothing was written.

### Change detection

`sync` compares the desired state against what OBS currently holds **before** making any write call:

- **Project / package meta** — the managed fields (title, description, repositories) are compared as XML; OBS-managed fields (ACL entries, person/group/lock) are ignored.
- **Project config** — the raw string is compared after stripping leading/trailing whitespace.
- **`obs/` files** — each file's MD5 is compared to the MD5 returned by the OBS source directory listing. Only changed files are uploaded. Files present on OBS but absent locally are deleted. All uploads and deletions are committed as a single OBS source revision.

Every resource is always printed with its status (`+`/`~`/`=`/`-`). The `=` line is printed even when nothing changed. Use `--force` to bypass comparison and always write.

### `sync push [--force] [--dry-run] [--no-services] [--no-cache] [--non-recursive] [--project-only] [--branch-from PROFILE] [--skip-unchanged] [--report-json PATH] [-m MSG] [project] [package]`

Syncs local packaging files to OBS. For each target package, all ancestor projects (from root down) are created/updated first, then the package meta is applied, then source files are synced as a **single OBS source revision**. Services are run locally to produce the upstream source tarball and packaging artifacts; the `_service` file is **not** uploaded to OBS.

| Call form | Effect |
|---|---|
| `sync push` | Sync all packages under `root/` |
| `sync push <project>` | Sync all packages under the project (recursively) |
| `sync push <top-level-package>` | Sync a single package directly under `root/` |
| `sync push <project> <package>` | Sync a single package under the project |

Options:
- `--force` — bypass OBS conflict checks; always write meta and files regardless of diff.
- `--dry-run` — run local services and report what would be uploaded to OBS without writing. All OBS writes are skipped but the `+`/`~`/`=`/`-` output reflects what would change.
- `--no-services` — skip local service execution; upload `obs/` as-is.
- `--no-cache` — bypass both cache levels; always run obs_scm and manual services from scratch.
- `--non-recursive` — only sync packages directly under the specified project; do not descend into sub-projects.
- `--project-only` — only sync project configuration (meta and build config); skip all package syncing.
- `--branch-from PROFILE` — for each package unchanged since the given profile's last sync, upload only an `_aggregate` file that reuses pre-built binaries from that profile's OBS project instead of uploading sources. The branch profile may target a different OBS instance. After the initial changed/unchanged classification, a second phase queries OBS `_builddepinfo` and automatically promotes any additional packages whose build dependencies or dependents were promoted (bidirectional fixed-point propagation). The aggregate message format is `branch: <profile> (<source_project>/<package>)`.
- `--skip-unchanged` — plain pushes only (rejected with `--branch-from`): skip packages whose OBS revision comment records a clean sync from a git SHA with no changes since (package-directory commits, uncommitted edits, inherited macros.yaml). One API call per skipped package, or zero when the `.cache/sync_state/` manifest is warm. Packages whose `_service` has an upstream obs_scm tracking a moving ref (branch or no revision) are never skipped; `--force` disables skipping entirely. See "Reducing OBS API traffic" in `docs/PERCONA_OBS_TOOL.md`.
- `--report-json PATH` — write a JSON sync report (`rebuild_projects`, `promoted`, `skipped`, `head_sha`) consumed by the CI poll script via `OBS_SYNC_REPORT` to scope build monitoring to the projects the sync actually touched.
- `-m MSG` / `--message MSG` — commit message recorded in the OBS source revision. When omitted, a message is generated automatically: `sync: <branch>@<short-sha> (<remote_url> or <hostname>)`

### `--branch-from` decision process

When `--branch-from <profile>` is given, each package is individually evaluated: either an `_aggregate` is uploaded (reusing pre-built binaries from the branch profile's OBS project) or sources are uploaded normally. The decision is made by `_resolve_branch_decision`.

#### Branch project derivation

The corresponding branch OBS project is derived by substituting the current `rootprj` prefix with the branch profile's `rootprj`. For example, if the current project is `home:Admin:percona-test:ppg:staging:17` and the branch rootprj is `home:Admin:percona`, the branch project is `home:Admin:percona:ppg:staging:17`.

#### Primary path — git SHA comparison

1. Fetch the latest source revision comment from the branch OBS project for this package (`GET /source/<branch_project>/<package>/_history`).
2. Match it against the sync message pattern `sync: <branch>@<sha> (<detail>)`.
3. If the message matches and the detail does **not** start with `"local changes on"` (i.e. was synced from a pushed branch):
   - Call `git log` to check whether any commits touching `<package_path>` exist since `<sha>`.
   - **No commits** → aggregate (package unchanged). **Commits exist** → upload sources.

#### Fallback — content check

The content check is used when the revision comment cannot be trusted:
- No comment on the branch (new project, never synced)
- Comment doesn't match the `sync:` format (e.g. manual commit, older format)
- Sync message says `"local changes on <hostname>"` (HEAD was not pushed to any remote)

Content check (`_content_matches_branch`) performs two sub-checks:

**Sub-check 1 — File MD5 comparison**

Fetch the expanded file list from OBS (`GET /source/<branch_project>/<package>?expand=1`). The `expand=1` parameter is required to see service-generated files (e.g. `_service:obs_scm:*.obsinfo`, `.obscpio`) that OBS stores server-side. For every file in the local `obs/` directory, compare its MD5 against the OBS-returned MD5. If any file differs or is missing from OBS, the check fails (→ upload sources).

**Sub-check 2 — Upstream obs_scm commit hash**

If a `_service` file exists, extract the *upstream* `obs_scm` service — the one that fetches the actual software source. Packaging `obs_scm` services (whose `subdir` param matches `root/.+/(debian|rpm)$` or equals `${DEBIAN_PACKAGE_DIRECTORY}` / `${RPM_PACKAGE_DIRECTORY}`) are excluded. If exactly one upstream `obs_scm` remains:

1. Resolve the remote HEAD SHA using `git ls-remote --` (30 s timeout), trying `refs/heads/<revision>`, then `refs/tags/<revision>^{}` (annotated tag), then `refs/tags/<revision>`.
2. If resolution fails, treat the package as **changed** (conservative: cannot verify).
3. Find the obsinfo file on OBS by looking for a name that starts with `<filename_prefix>` or `_service:obs_scm:<filename_prefix>` and ends with `.obsinfo`. OBS stores server-side service outputs with a `_service:<name>:` prefix; both forms are checked.
4. Fetch the obsinfo content with `?expand=1` and parse the `commit:` line.
5. If `obs_commit != remote_head_sha` → **changed** (upstream has moved). If equal → **unchanged**.

If zero or more than one upstream `obs_scm` services are found, sub-check 2 is skipped and the MD5 match alone is sufficient.

#### Phase 2 — Build dependency propagation

After Phase 1 classifies every package as `"aggregate"`, `"skip_branch"`, or
`"promote"`, Phase 2 enforces build dependency correctness by promoting any package
whose build dependencies or dependents have been promoted.

**Why this is necessary**: if package **A** (e.g. `golang-1.25`) has local changes and
is promoted, packages that build-depend on A (e.g. `percona-telemetry-agent`, `etcd`)
must also be promoted — otherwise they would link against the old branch binaries and
not the new A. Conversely, packages that A depends on are also promoted so A builds
against locally-controlled sources rather than the branch copy.

**How it works**:
1. Determine which OBS projects to query for `_builddepinfo`:
   - With `--branch-from`: query the **branch OBS** (`branch_apiurl`) for all branch
     projects derived from every package in scope (not just the ones with `"aggregate"`
     decisions), including e.g. `<branch_rootprj>:builddep`.
   - Without `--branch-from` (plain push over a previously branched env): query the
     **target OBS** (`apiurl`) for the union of target projects and any source projects
     recorded from `branch:` revision comments (`branch_project_for.values()`).
2. Call `_fetch_combined_depinfo(dep_apiurl, dep_projects, local_pkg_names)` to build:
   - `providers_by_project`: per-project map of binary package name → source OBS
     package (multibuild `:flavor` suffixes are stripped from source names at
     construction time); each consumer's dep is resolved own-project-first, then
     via the queried repo's `<path>` chain.
   - `fwd_deps[A]`: set of local packages that **A** build-depends on.
3. Run fixed-point bidirectional propagation until stable:
   - **Forward**: if B is in `fwd_deps[A]` and B is promoted → promote A.
   - **Backward**: if A is promoted and B is in `fwd_deps[A]` → promote B.
4. All packages whose decision was changed to `"promote"` by Phase 2 log a message
   indicating which dep triggered them.

#### Phase 2.5 — Project config change detection and config-triggered promotion

After dependency propagation, `sync push --branch-from` checks whether each in-scope project's
desired metadata (repositories, build flags, `project-config`) differs from what is currently on
OBS — and promotes all packages in that project if a difference is detected.

**Why this is necessary**: a PR may change only `root/project.yaml` (e.g. adding a new architecture
like `aarch64`). No package files change, so Phase 1 assigns `"aggregate"` to everything and Phase 2
propagates nothing. Phase 2.5 detects the config change and upgrades every package in the affected
projects to `"promote"`, causing them to be built from source on the new architecture.

**How it works**:

1. For every project in the PR namespace, call `check_project_config_changed(apiurl, pr_project, ...)`.
   This builds the locally-desired project meta XML and compares it against the live OBS meta.

2. The function returns `(changed: bool, is_new: bool)`.
   - `is_new=False, changed=True` → project already existed on OBS and its config changed → add to
     `config_changed_projects` and promote all packages.
   - `is_new=True` (PR project does not yet exist) → fallback comparison:
     - Derive the corresponding production project name (the branch source project under
       `branch_rootprj`).
     - Call `check_project_config_changed(branch_apiurl, prod_project, ...)` to compare the local
       desired config against what the production project currently has on the branch OBS.
     - If `not is_new and changed` in that comparison (production exists and its config differs from
       local) → return `(changed=True, is_new=False)` so the project enters `config_changed_projects`.
   - Otherwise → no config-triggered promotion for this project.

3. Projects in `config_changed_projects` force all their packages to `"promote"`, and those projects
   are included in `active_projects` so they are created on the PR OBS.

**Why the fallback is safe**: for a re-sync of an existing PR project `is_new=False`, so the
fallback is never reached. For a new PR where no inherited config changed, the fallback compares
local vs production and finds them equal → no spurious promotions. Only when an arch (or other
config field) was genuinely added does the fallback produce a promotion trigger.

#### Plain `sync push` with a `branch:` aggregate already on OBS

When running `sync push` *without* `--branch-from` (i.e. a full source sync), but the package on OBS already holds a `branch:` aggregate from a previous `--branch-from` run, uploading sources would overwrite the aggregate unnecessarily. To detect this:

1. Fetch the latest revision comment.
2. If it matches `branch: <profile> (<source_project>/<package>)`, extract `<source_project>`.
3. Run the content check (`_content_matches_branch`) against that source project.
4. If the content matches → print `= files ...` and skip the upload. If not → proceed with the normal source upload.

#### Multibuild packages

For packages with an `obs/_multibuild` file, the `_aggregate` XML must list every flavored OBS package name separately. `_multibuild_packages(obs_dir, base_name)` reads the `<flavor>` elements and checks the `buildemptyflavor` attribute (default: true). When `buildemptyflavor` is absent or `"true"`, the bare package name is included in addition to `<base_name>:<flavor>` entries. The `_aggregate` output format is:

```xml
<aggregatelist>
  <aggregate project="<branch_project>">
    <package>percona-pg-telemetry:17</package>
    <!-- <package>percona-pg-telemetry</package>  only if buildemptyflavor != false -->
  </aggregate>
</aggregatelist>
```

The revision message recorded for the aggregate commit is `branch: <profile> (<branch_project>/<package>)`.

### `sync delete [--yes] [--recursive] [--dry-run] [project] [package]`

Deletes OBS projects (and their sub-projects) or a single package created by `sync push`.

| Call form | Effect |
|---|---|
| `sync delete` | Delete the full project tree under rootprj (deepest sub-projects first) |
| `sync delete <project>` | Delete a project and all its sub-projects |
| `sync delete <project> <package>` | Delete a single package |

Options:
- `--yes` / `-y` — skip the confirmation prompt.
- `--recursive` — delete projects that still contain packages (passes OBS `recursive` flag). Without this flag, a project with packages will fail with a hint to add `--recursive`.
- `--dry-run` — show what would be deleted without making any changes.

Projects that do not exist on OBS are silently skipped. Projects are always deleted with `force=True` to bypass inter-project repository dependency checks when removing a whole tree.

### `sync release-pr [--yes] [project]`

Copies binaries from a PR OBS project to the corresponding production project using `osc release`,
then updates the production project configs to match the local desired state. Used to merge a
non-release PR (one that does not create a version tag) once its OBS builds pass.

**How it works**:

1. Walk the PR project tree (under `rootprj`) and call `osc release` for every package in every
   PR sub-project that exists on OBS. `osc release` copies binary packages from the PR project into
   the production project (identified by the `<releasetarget>` in each repository's OBS meta).

2. After releasing, discover which production projects were targeted by reading the
   `<releasetarget project="...">` elements from each PR project's OBS meta.

3. For each discovered production project, call `_apply_project_config()` to sync the locally
   desired project meta to the production OBS — this propagates any config changes (e.g. a newly
   added architecture) that were part of the PR but are not yet in the production project.
   The `env_vars` dict for this step is seeded from the active profile env overrides (so
   `${REMOTE_OBS_ORG_INTERCONNECT}` and similar vars are available) plus
   `OBS_ROOTPRJ=<production_rootprj>`.

**Important**: `sync release-pr` only releases binaries; it does not delete the PR project. Run
`sync delete --yes --recursive` afterward (as `obs-pr-cleanup.yml` does) to clean up.

Options:
- `--yes` / `-y` — skip the confirmation prompt.

### `sync promote [--dry-run] [--no-services] [--no-cache] [-m MSG] [project] [package]`

Promotes branch packages (created by a prior `--branch-from` sync) back to full source syncs. For each targeted package whose latest OBS revision comment matches the `branch:` pattern, the `_aggregate` is replaced with the local `obs/` source files (running any `mode="manual"` services as needed). Packages that already hold real sources are skipped (`=` output).

| Call form | Effect |
|---|---|
| `sync promote` | Promote all branch packages under rootprj |
| `sync promote <project>` | Promote all branch packages under the project |
| `sync promote <project> <package>` | Promote a single package |

Detection: reads the latest OBS revision comment via `_fetch_obs_package_latest_comment`; if it matches `_BRANCH_MSG_RE` (`^branch: \S+ \((.+)/[^/]+\)$`), the package is a branch and will be promoted. Packages without an `obs/` directory are silently skipped.

Options:

- `--dry-run` — show what would be promoted without writing to OBS. Services are not run in dry-run mode.
- `--no-services` — upload `obs/` files as-is without running manual services.
- `--no-cache` — disable the service artifact cache.
- `-m`/`--message` — OBS revision commit message (defaults to the standard sync message).

### Local service execution

If a package's `obs/_service` contains any service with `mode="manual"`, `sync` automatically runs all non-buildtime services locally before uploading. This is required for packages like Go services that use `go_modules` (mode=manual) to vendor dependencies.

**Execution order and file handling:**
1. All services with `mode` not in `{buildtime, serveronly, disabled}` are run in XML declaration order.
2. Each service binary is invoked from `/usr/lib/obs/service/<name>` with its `<param>` values and `--outdir`.
3. Service outputs are merged into a shared work directory so later services can consume earlier outputs (e.g. `go_modules` consuming `obs_scm` tarballs).
4. Only files produced by `mode="manual"` services are committed to OBS. Files produced by no-mode services (e.g. obs_scm source tarballs) are used locally but **not** uploaded — OBS regenerates those on its server.

If a service binary is missing from `/usr/lib/obs/service/`, a warning is logged and the service is skipped. A non-zero service exit code aborts the entire `sync` run.

#### Local service cache

To avoid re-running expensive operations (git clones, Go/Rust dependency vendoring, tarball downloads) on every `sync`, `percona-obs` maintains a four-level on-disk cache at `.cache/` in the project root (git-ignored via `.gitignore`).

**Level 1 — obs_scm output cache** (`.cache/obs_scm/{params_hash}/{head_sha}/`)

Before invoking each `obs_scm` service binary, `percona-obs`:
1. Computes `params_hash` as the SHA256 of all sorted `name=value` param pairs from the service XML element. Any change to the service config (URL, revision, extract pattern, etc.) produces a different key.
2. Calls `git ls-remote` (30 s timeout) to resolve the remote revision to a commit SHA (`head_sha`), trying in order: `refs/heads/<revision>`, `refs/tags/<revision>^{}` (annotated tag, peeled to commit), `refs/tags/<revision>`.
3. Checks `.cache/obs_scm/{params_hash}/{head_sha}/`. On a **hit**, all cached files are restored to the work directory and obs_scm is skipped entirely. On a **miss**, obs_scm runs normally and its output files (`.obsinfo`, `.obscpio`, `.dsc`, etc.) are stored atomically to `.cache/obs_scm/{params_hash}/{head_sha}/`.

If `git ls-remote` fails or times out, obs_scm always runs and its output is not stored.

**Level 2 — manual service output cache** (`.cache/services/{upstream_commit}/`)

After Phase 1 completes, `percona-obs` identifies the *upstream source* `obs_scm` service — the one that fetches the actual software being packaged — by filtering out every `obs_scm` whose `subdir` param matches `root/.+/(debian|rpm)$` or equals `${DEBIAN_PACKAGE_DIRECTORY}` / `${RPM_PACKAGE_DIRECTORY}` (those fetch packaging files from this repo). Exactly one service must remain; zero or two or more trigger a warning and the cache is skipped.

The obsinfo file produced by that upstream obs_scm is named `{filename}.obsinfo` (where `filename` is the service's `filename` param, e.g. `etcd.obsinfo`). Its `commit:` field — the HEAD commit of the upstream repo at fetch time — is used as the cache key.

- **Cache hit**: `.cache/services/{upstream_commit}/` exists and contains files → those files (vendor tarballs, etc.) are copied to the work directory, all `mode="manual"` services are skipped, and the function returns immediately.
- **Cache miss**: all `mode="manual"` services run in XML-declaration order, then their output files are stored atomically to `.cache/services/{upstream_commit}/`.

**Level 3 — download_url output cache** (`.cache/download_url/{params_hash}/`)

Before invoking each `download_url` service binary, `percona-obs` computes `params_hash` as the SHA256 of all sorted `name=value` param pairs from the service XML element (after macro/env substitution). Since the URL fully determines the downloaded content for the versioned artifacts these services fetch, no remote check is performed. On a **hit**, the cached files are restored to the work directory and the download is skipped. On a **miss**, `download_url` runs normally and its output files are stored atomically to `.cache/download_url/{params_hash}/`.

**Level 4 — cargo_vendor output cache** (`.cache/cargo_vendor/{params_hash}/{source_id}/`)

`cargo_vendor` is declared `mode="buildtime"` but is not a fast local transform — it downloads the full crate dependency tree from crates.io. Its output (`vendor.tar.gz`) is cached, keyed on `params_hash` (SHA256 of the service params) plus `source_id`, which identifies the exact source being vendored: the upstream obs_scm commit hash when the package has an obs_scm service, otherwise the SHA256 of the resolved `src` archive(s) themselves (for sources fetched via download_url). Any upstream commit or source change therefore produces a new key and invalidates the cache; on store, entries for older revisions of the same service are pruned (vendor tarballs are large). If no `source_id` can be determined, cargo_vendor runs uncached.

**Atomic writes**: all levels write to a temporary directory inside the cache directory (ensuring same filesystem), then rename it into place, preventing partial or corrupt cache entries.

**`--no-cache`**: pass to `sync` to bypass all cache levels unconditionally for that run.

When targeting a specific package (`sync <project> <package>`), the ancestor project chain is only walked if the target project does not yet exist on OBS (fast path avoids redundant GET calls otherwise).

Project names use colon notation matching the directory hierarchy (e.g. `ppg:staging:17`).

### `build trigger [project] [package]`

Triggers an OBS service run (`runservice`) for one or more packages, causing OBS to re-fetch sources and rebuild.

| Call form | Effect |
|---|---|
| `build trigger` | Trigger services for all packages under `root/` |
| `build trigger <project>` | Trigger services for all packages under the project |
| `build trigger <top-level-package>` | Trigger service for a single top-level package |
| `build trigger <project> <package>` | Trigger service for a single package under the project |

### `build status [project] [package]`

Prints a color-coded tree of live build statuses fetched from OBS. For each repository where a package has `succeeded`, the built version (e.g. `3.5.26-6.1`) is shown after the status symbol, parsed from the binary package filename.

| Call form | Effect |
|---|---|
| `build status` | Status for all packages under `root/` |
| `build status <project>` | Status for all packages under the project (tree rooted there) |
| `build status <top-level-package>` | Status for a single top-level package |
| `build status <project> <package>` | Status for a single package |

Status symbols (color output disabled with `NO_COLOR=1`):

| Symbol | Color | OBS status codes |
|---|---|---|
| `✔` | green | `succeeded` |
| `✗` | red | `failed` / `unresolvable` / `broken` |
| `●` | cyan | `building` / `dispatching` |
| `◌` | yellow | `scheduled` / `blocked` |
| `–` | dim | `excluded` / `disabled` |
| `?` | dim | `unknown` or any unrecognised code |

For multibuild packages, when all flavors of a repository share the same status the flavor tags are shown inline (e.g. `[:17]`). When flavors differ, each expands to its own sub-line under the repository.

When multiple architectures are configured for the same repository, the highest-priority (most actionable) status is kept per flavor; arch details are not shown.

### `build dependency [project]`

Queries OBS `_builddepinfo` for all packages in scope and prints a build dependency
tree. Packages are grouped by **root packages** (packages that no other local package
depends on). Each root package is a tree root; its direct and transitive build
dependencies are indented beneath it with box-drawing characters.

| Call form | Effect |
|---|---|
| `build dependency` | Dependency tree for all packages under `root/` |
| `build dependency <project>` | Restrict to packages under the given project |

**Output format**: each line is `<pkg> (<obs_project>)`. Root packages (tree roots) are
printed in bold. Packages with no local dependencies and nothing depending on them are
listed after all trees as isolated packages. Cycles are detected and printed as
`(cycle)` leaf nodes.

**Implementation** (`cmd_build_dependency` in `cmd_build.py`):
1. Scan all packages under scope with `find_packages`.
2. Collect all OBS project names from those packages.
3. Call `_fetch_combined_depinfo(apiurl, dep_projects, local_pkg_names)` to build
   `fwd_deps` (source package → set of local packages it depends on), resolving
   each binary to its provider project-aware (own project, then repo path chain).
4. Identify root packages: any package **not** present in any `fwd_deps` value set.
5. Print trees with `_print_dep_tree()`, then isolated packages (no deps, not depended
   on by anything).

### `project verify [project] [-P <profile>] [-e KEY:VALUE ...]`

Validates local project configuration without connecting to OBS.

The optional `project` argument (colon notation, e.g. `ppg:staging:17`) restricts validation to that subtree. If omitted, the entire `root/` tree is validated.

**Check 1 — subproject references**: every `subproject:` entry in all `project.yaml` files within the scope must resolve to an existing directory under `root/`.

**Check 2 — env variable coverage**: every `${VAR}` token found in `project.yaml`, `package.yaml`, and `obs/_service` / `obs/_aggregate` / `obs/_link` files within the scope must be defined in the active env.

Env resolution for the check (same precedence as all other commands):
- Profile env (`-P <profile>`) provides the base values.
- `-e KEY:VALUE` flags override or supplement individual variables.
- With no profile and no `-e` flags, any `${VAR}` token found is an error with a hint to supply a profile.

```sh
# Validate the entire tree against the dev profile
./percona-obs -P dev project verify

# Validate only the ppg:staging:17 subproject
./percona-obs -P dev project verify ppg:staging:17

# Check with an inline override (no profile file needed)
./percona-obs -e REMOTE_OBS_ORG_INTERCONNECT:'openSUSE.org:' project verify
```

Exit code is 0 on success, 1 if any check fails.

## Maintaining Release Changelogs

Release changelogs live at `root/ppg/releases/<major>/CHANGELOG.md` and follow [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

### Rules for release note URLs

Every changelog entry that references an upstream version **must** include a URL to the official upstream release notes or changelog. When adding or fixing entries:

1. **Always verify the URL exists** — search the internet for the correct upstream release notes page. Do not guess or use a generic repo URL.
2. **Follow the established URL pattern per package** (see table below). Adapt the pattern to the new version rather than copying a raw git URL or a template artifact.
3. **Template artifacts** (`%!{VAR}`) are build-time errors left by the changelog generator — always replace them with the resolved URL.
4. If no established pattern exists for a package, search for its official release notes page and document the pattern in the table below.

### URL patterns by package

| Package | URL pattern | Example |
|---|---|---|
| `percona-postgresql` | `https://www.postgresql.org/docs/release/<version>/` | `…/release/17.10/` |
| `etcd` | `https://github.com/etcd-io/etcd/releases/tag/v<version>` | `…/tag/v3.5.30` |
| `percona-haproxy` | `https://www.haproxy.org/download/<major.minor>/src/CHANGELOG` | `…/2.8/src/CHANGELOG` |
| `percona-patroni` | `https://github.com/zalando/patroni/releases/tag/v<version>` | `…/tag/v4.1.3` |
| `percona-pg_gather` | `https://github.com/jobinau/pg_gather/releases/tag/v<version>` | `…/tag/v33` |
| `percona-pg_tde` | `https://github.com/percona/pg_tde/releases/tag/<version>` | `…/tag/2.2.0` |
| `percona-pgbouncer` | `https://github.com/pgbouncer/pgbouncer/releases/tag/pgbouncer_<version_with_underscores>` | `…/pgbouncer_1_25_2` |
| `percona-pgpool-II` | `https://www.pgpool.net/docs/<major.minor>/en/html/release-<version-with-dashes>.html` | `…/release-4-7-1.html` |
| `percona-postgis` | `https://github.com/postgis/postgis/blob/<version>/NEWS` | `…/blob/3.5.6/NEWS` |
| `percona-postgresql-common` | `https://salsa.debian.org/postgresql/postgresql-common/-/tags/debian%2F<version>` | `…/debian%2F290` |
| `percona-telemetry-agent` | `https://github.com/percona/telemetry-agent/releases/tag/v<version>` | `…/tag/v1.0.13` |

## Adding a New PostgreSQL Extension
1. Copy `ppg/staging/17/percona-pg-telemetry/` as a template
2. Replace all `percona-pg-telemetry` references with the new package name
3. Update `obs/_multibuild` flavors for the target PG versions
4. Update `obs/_service` upstream URL to point to the new package's GitHub repo
5. Update `rpm/*.spec` — preserve `@BUILD_FLAVOR@` in `Name:` and `%define pg_version`
6. Update `debian/control` — keep `@BUILD_FLAVOR@` in `Package:` and version-specific `Depends:`

## Adding a New Standalone Service (Go)
1. Copy `percona-telemetry-agent/` as a template
2. Update `obs/_service` upstream URL; keep `go_modules` service in manual mode
3. `debian/rules` version extraction pattern reads `/usr/src/packages/SOURCES/*.obsinfo`
4. Ensure `vendor.tar.gz` is listed in `debian/debian.dsc`'s `Debtransform-Files-Tar`

## Importing an Existing OBS Package

When given an OBS package URL and a target location within `root/`, follow these steps. The user may request either a **full source import** (copy all files from OBS) or an **aggregate import** (create an `_aggregate` link so the local OBS pulls built packages from the source project). Use the mode explicitly requested; default to full source import if not specified.

### Inputs
- **OBS package URL** — the web UI URL, e.g. `http://192.168.1.103:3000/package/show/home:Admin/obs-service-tar_scm`
- **Target location** — directory relative to `root/` where the package should land (e.g. `root/` for a top-level package, `root/ppg/staging/17/` for a subproject package)
- **Import mode** — `full` (copy source files) or `aggregate` (create `_aggregate` link)

### Step 1 — Determine the API URL

The web UI URL and API URL are not always the same host:

| Web UI host | API host to use |
|---|---|
| `build.opensuse.org` | `api.opensuse.org` |
| Any other host | same host as the web UI |

The path format `/package/show/<project>/<package>` always identifies the OBS project and package name regardless of which host is used.

### Step 2 — Fetch package metadata

```sh
osc -A <apiurl> api /source/<obs_project>/<package_name>/_meta
```
Extract `<title>` and `<description>`. Treat a description containing only whitespace as empty.

### Step 3 — Create the directory

```sh
mkdir -p root/<target>/<package_name>/obs
```

### Step 4a — Full source import

List files:
```sh
osc -A <apiurl> api /source/<obs_project>/<package_name>
```
Download each `<entry name="...">`:
```sh
osc -A <apiurl> api /source/<obs_project>/<package_name>/<filename>
```
Place every file directly in `obs/`. Do **not** split into `debian/` or `rpm/` — that is a separate step if desired.

### Step 4b — Aggregate import

Create `obs/_aggregate` pointing to the source project. When the source is on a remote OBS instance, prefix the project name with the instance identifier:

| Source OBS instance | Project name in `_aggregate` |
|---|---|
| `build.opensuse.org` / `api.opensuse.org` | `openSUSE.org:<obs_project>` (e.g. `openSUSE.org:openSUSE:Tools`) |
| Local OBS (`192.168.1.103`) | Use the project name as-is (e.g. `home:Admin`) |

```xml
<aggregatelist>
  <aggregate project="<mapped_project>">
    <package><package_name></package>
  </aggregate>
</aggregatelist>
```

### Step 5 — Write `package.yaml`

```yaml
title: "..."        # quote if the value contains a colon
description: |
  <description from _meta, reflowed to ~80 chars per line>
```
Omit `package.yaml` entirely if both `<title>` and `<description>` are empty.

**Always quote the `title` value with double quotes if it contains a colon (`:`) — YAML treats an unquoted colon as a mapping separator and will fail to parse.**

### Notes
- Use the OBS package name unchanged as the local directory name.
- Do not run `black` or `pyright` — no Python code is modified.
- After creating the files, verify with `find root/<package_name> -type f | sort`.

## OBS Package Branching Mechanisms

Source code reference: `/home/rdias/Work/open-build-service/` — key files: `src/api/app/models/branch_package.rb`, `src/api/app/controllers/source_package_command_controller.rb`, `src/backend/BSSrcServer/Link.pm`, `src/backend/BSSched/BuildJob/Aggregate.pm`.

| Mechanism | Creates `_link`? | Independent? | Follows devel chain? | Use case |
|---|---|---|---|---|
| `_link` file | yes (manually) | no | no | overlay/patch tracking |
| `cmd=branch` | yes (auto) | no | yes | developer workflow |
| `cmd=fork` | no (scmsync) | yes (git) | yes | SCM-based development |
| `cmd=copy` | no | yes | no | release, snapshot |
| `cmd=linktobranch` | transforms | partially | no | patch a linked package |
| `_aggregate` | no | n/a | no | binary reuse across projects |

### 1. `_link` — Source Link (core primitive)

A `_link` XML file inside a package makes it inherit sources from another package. The backend (`BSSrcServer/Link.pm`) resolves the link chain at build time — it fetches the origin's files, applies any local overlays, and presents the merged filelist to the build system. Links can chain. `rev`/`srcmd5` can pin a specific revision.

```xml
<link project="BaseProject" package="mypackage" rev="abc123"/>
```

### 2. `cmd=branch` — Package Branch

`POST /source/<project>/<package>?cmd=branch&target_project=<tgt>`

The main "developer branch" operation (`BranchPackage` in `branch_package.rb`):
1. Creates a branch project (e.g. `home:user:branches:BaseProject`)
2. Creates a package in it with a `_link` pointing back to the source
3. Optionally follows the **devel project** chain (`devel:` pointer on the package)
4. Optionally follows the **update project** chain (`OBS:UpdateProject` attribute)
5. Copies repositories from the source project

Resolution order:
```
1) BaseProject  ←  2) UpdateProject  ←  3) DevelProject/Package
                                          X) BranchProject  ← branch targets here
```

Key parameters: `maintenance=1`, `newinstance=1` (copy instead of link), `ignoredevel=1`, `missingok=1`, `dryrun=1`.

### 3. `cmd=fork` — SCM-synced Fork

`POST /source/<project>/<package>?cmd=fork&scmsync=<url>`

Variant of `branch` for `scmsync` (Git-managed) packages. Creates a new package with its own `scmsync` URL pointing to a forked repo. Same `BranchPackage` code path but skips all source link operations.

### 4. `cmd=copy` — Full Source Copy

`POST /source/<project>/<package>?cmd=copy&oproject=<src>&opackage=<src_pkg>`

A complete, independent copy of source files — no `_link`. The new package is fully independent of the origin. Used for releases, snapshots, and starting a new independent package from an existing one. Key options: `keeplink=1`, `expand=1`, `repairlink=1`, `withvrev=1`.

### 5. `cmd=linktobranch` — Convert Link to Branch

`POST /source/<project>/<package>?cmd=linktobranch`

Converts an existing `_link` package into a proper branch (expands the link, stores real files, keeps the link with a baserev). Useful when you need to make actual changes to a linked package.

### 6. `_aggregate` — Binary Aggregation

A special package type (`_aggregate` XML file) that pulls **built binaries** (not sources) from another project's repository into the current one. Handled by `BSSched/BuildJob/Aggregate.pm`. No source link involved — the binaries are made available as if they were built locally.

### `osc` Python API

```python
import osc.core
osc.core.branch_pkg(apiurl, src_project, src_package, ...)   # cmd=branch
osc.core.copy_pac(src_apiurl, src_project, src_package, ...)  # cmd=copy
osc.core.link_to_branch(apiurl, project, package)             # cmd=linktobranch
```

## Direct OBS CLI (osc)
```sh
# Check out a package from OBS
osc co <project> <package>

# Sync local files into the OBS checkout, then commit
cp -r obs/* <checkout>/
osc add <new-files>
osc ci -m "update _service"

# Trigger a remote rebuild
osc rebuild <project> <package>

# Follow build log
osc buildlog <project> <package> <repo> <arch>
```

## Debugging and Bug Investigation Workflow

When investigating a bug or unexpected behaviour in `percona-obs`, use the following iterative process.

### 1. Reproduce with `--verbose --dry-run`

Always start with a fully-verbose, non-destructive run:

```sh
./percona-obs --verbose -P <profile> sync push [args] --dry-run
```

- `--verbose` enables `DEBUG`-level log messages (prefixed `  · `), showing every OBS API call, unchanged-item decisions, dep-promotion steps, and cache hits/misses.
- `--dry-run` runs local services and reports what would change on OBS without writing anything.

For `build dependency`, which has no dry-run flag, just run it directly against the test profile:

```sh
./percona-obs --verbose -P <profile> build dependency
```

### 2. Read the verbose output carefully

Key things to look for in verbose output:

| Verbose line | What it tells you |
|---|---|
| `planning: checking sync decisions` | Phase 1 started |
| `branch decision: git-log  <pkg>  (…)` | Git-based unchanged check for a package |
| `branch decision: content check  <pkg>  (…)` | MD5/obsinfo fallback check for a package |
| `branch decision: aggregate  <pkg>  (content matches)` | Package was classified as aggregate |
| `planning: checking build dependencies (N project(s))` | Phase 2 started; N = number of projects queried |
| `dep-promote: builddepinfo covers N local packages` | How many packages the dep query returned data for |
| `dep-promote: <pkg> promoted by dep on <other>` | A package was cascade-promoted by Phase 2 |
| `_resolve_provider: ambiguous providers for <binary>` | A dep edge was dropped: multiple projects build the binary and none is in the consumer's repo path chain |
| `fetching revision history: <project>/<pkg>` | OBS revision comment lookup (build dependency / Phase 1 plain-push) |

If `dep-promote: builddepinfo covers 0 local packages` appears, the dep query returned nothing — likely querying the wrong OBS instance or querying projects with no build results yet.

Dep edges attribute each binary to a provider by searching the consumer's own
project first, then the `<path>` projects of the queried repository (fetched
from the project `_meta`), then falling back to the unique provider across all
queried projects.  Same-named binaries in sibling tiers (devel vs staging of
the same PG major) are therefore never conflated.

### 3. Add targeted `logger.debug()` calls

When verbose output is insufficient, add temporary debug logging to the relevant function. The `logger` instance is available in every module via `from .common import logger`. Example:

```python
logger.debug(f"dep-promote: dep_projects={dep_projects!r}, dep_apiurl={dep_apiurl!r}")
logger.debug(f"branch decision raw comment: {prior_comment!r}")
```

Run again with `--verbose` to see the new output. Remove the temporary lines once the root cause is found.

### 4. Check the OBS source revision comment directly

The revision comment on a package is the key data used by Phase 1 and `build dependency` to detect aggregates. Inspect it with:

```sh
osc -A <apiurl> api /source/<project>/<package>/_history | tail -20
```

The comment format must match one of these patterns (see `_SYNC_MSG_RE` / `_BRANCH_MSG_RE`):
- `sync: <branch>@<sha> (<detail>)` — normal source sync
- `sync: <branch>@<sha> (local changes on <hostname>)` — dirty sync
- `branch: <profile> (<source_project>/<package>)` — aggregate from `--branch-from`

If the comment doesn't match either pattern, `percona-obs` treats the package as changed (promotes it).

### 5. Check `_builddepinfo` directly

When Phase 2 or `build dependency` is not finding expected deps, verify what OBS actually returns:

```sh
osc -A <apiurl> api /build/<project>/_builddepinfo
```

Key things to check:
- Is the project listed at all? (It won't be if all packages are aggregates — `_builddepinfo` only covers packages with real build results.)
- Are the package names in `<package name="…">` matching what you expect? Multibuild packages appear as `pkg:flavor` — the code strips the `:flavor` suffix.
- Do the `<pkgdep>` entries resolve to binary names that appear as `<subpkg>` in another package's entry?

### 6. Trace the cross-instance routing

When branching is involved, always confirm which OBS instance is being queried:

- **`sync push --branch-from <profile>`**: Phase 2 always queries `branch_apiurl` (from the branch profile). Verify that the branch profile's `apiurl` field is set correctly with `./percona-obs profile list`.
- **`sync push` (plain, over a previously branched env)**: Phase 2 loads the profile named in each `branch:` revision comment to get its `apiurl`. If the profile has been renamed or deleted, it falls back to the target `apiurl` with a debug log.
- **`build dependency`**: checks every package's revision comment individually; routes each package's source project to the correct `apiurl` via the same profile lookup.

### 7. Common root causes seen in practice

| Symptom | Root cause | Fix applied |
|---|---|---|
| Phase 2 reports "0 local packages" | Querying target OBS for projects that have only aggregates (no build results) | Query the branch OBS (`branch_apiurl`) instead of the target OBS |
| Dep propagation not triggering for `pkg:flavor` multibuild | OBS `_builddepinfo` names source as `pkg:flavor`; lookup uses base name | Strip `:flavor` suffix in `_fetch_combined_depinfo` |
| Plain push does not cascade-promote dependents | `branch_project_for` not populated for "promote" decisions | Always record `branch_project_for[key]` when a `branch:` comment is detected, regardless of decision |
| `build dependency` missing deps for aggregate packages | Querying target OBS (aggregates have no builddepinfo there) | Check revision comment per package; route aggregate packages to their source OBS |
| `build dependency` wrong for mixed projects (some promoted, some aggregate) | Single-representative heuristic picked the promoted package | Check ALL packages (not just a representative) to cover mixed projects |
| `_validate_project_path_refs` hangs on external/interconnect OBS | Querying live OBS for non-local project references | Build local project name whitelist from `find_projects(REPO_ROOT, …)`; skip any `project:` entry not in that set |

## Key Files by Pattern
| Purpose | Exemplar |
|---|---|
| Go standalone package | `percona-telemetry-agent/` |
| PG extension multi-version | `ppg/staging/17/percona-pg-telemetry/` |
| Large PG server package | `ppg/staging/17/percona-postgresql17/` |
| Third-party infrastructure service | `ppg/staging/17/etcd/` |
| OBS aggregate (mirrors another OBS project) | `obs-service-tar_scm/` |
| Root project config | `root/project.yaml` |
| Management script | `percona-obs` (commands: `sync push`, `sync delete`, `sync promote`, `build trigger`, `build status`, `build dependency`, `profile create`, `profile list`, `project verify`) |

---

## GitHub Actions CI/CD

Four workflows automate the OBS sync lifecycle. All of them use a shared composite action for setup.

### Composite action — `.github/actions/obs-setup`

Reusable setup steps called by every workflow:
1. Install Python 3.11 via `actions/setup-python`.
2. Create `venv/` and `pip install -r requirements.txt`.
3. Write `~/.config/osc/oscrc` from the `obs-apiurl`, `obs-user`, and `obs-password` inputs so `osc` (and therefore `percona-obs`) can authenticate against OBS.

### Workflow 1 — `sync-main.yml` (sync on merge to main)

**Trigger**: push to `main` where at least one file under `root/**` changed.

**What it does** — two jobs. The `sync` job serializes on its own concurrency group and is never cancelled mid-upload; the `poll` job runs in a cancel-superseding group (newest poll wins — version lists and release tags are diffed from the last *successful* run's head SHA, so the surviving poll covers superseded runs' ranges).

`sync` job:
1. Checks out the repo with full history (`fetch-depth: 0`) — required because `sync push` reads `git log` to detect per-package changes since the last OBS sync SHA stored in OBS revision comments.
2. Runs `obs-setup` and restores the `.cache/` actions/cache entries (including the `sync_state` manifest used by `--skip-unchanged`).
3. Creates a `percona-obs` profile named `main` pointing at `OBS_ROOTPRJ` with env vars `PERCONA_OBS_PACKAGING_BRANCH=main`, `PERCONA_OBS_PACKAGING_REPO=<repo-url>`, and `REMOTE_OBS_ORG_INTERCONNECT=` (empty — no interconnect in the self-hosted setup).
4. Runs `percona-obs -P main sync push --no-scm-validate --skip-unchanged --report-json /tmp/sync-report.json` to create/update OBS projects and packages, delete any OBS packages whose local directories were removed, and hand the sync report to the poll job as an artifact.

`poll` job:
5. Runs `.github/scripts/poll_obs_builds.py` to poll `build status` until all packages reach a terminal state (`succeeded`/`failed`/`unresolvable`/`disabled`). `OBS_SYNC_REPORT` points at the sync report so monitoring is scoped to the projects the sync actually touched (failing open to full-tree polling on a missing/corrupt/stale report); the poll interval backs off ×1.5 while states are unchanged (`OBS_POLL_INTERVAL`, default 30 s, up to `OBS_POLL_MAX_INTERVAL`, default 300 s). The script exits non-zero when any build failed/broke/was unresolvable, which fails the job's GitHub Actions check run — that check run is the merge-gating signal, no separate commit status is posted.

**Required repository config**: `OBS_APIURL`, `OBS_WEB_URL`, `OBS_ROOTPRJ`, `OBS_USER` (vars); `OBS_PASSWORD` (secret).
Permissions: `contents: write` (for badge publishing).

### Workflow 2 — `obs-pr-check.yml` (PR build check)

**Trigger**: `pull_request` against `main` (types: `opened`, `synchronize`, `reopened`, `labeled`) where at least one file under `root/**` changed. The sync/build (and QA) jobs only run when a trigger label — `obs-sync`, `qa-packages`, or `qa-containers` — is present on the PR. For `labeled` events, a job-level `if` on `resolve` skips the entire run unless the label just added is a trigger label; for the other event types `resolve` runs, checks the PR's labels, and if no trigger label is present the run gates out after `resolve` with the rest of the DAG skipped.

**What it does**:
1. Full-history checkout (same reason as above).
2. Runs `obs-setup`.
3. Creates **two** `percona-obs` profiles:
   - `main` — points at `OBS_ROOTPRJ`; used only as the `--branch-from` source so the tool can read each package's last-sync SHA from the main OBS project.
   - `pr-<N>` — points at `OBS_PR_ROOTPRJ:pr-<N>` (the PR-specific OBS root project). Its `PERCONA_OBS_PACKAGING_BRANCH` is set to `refs/pull/<N>/head` (a GitHub pseudo-ref resolvable via `git ls-remote` for all PRs including forks) and `PERCONA_OBS_PACKAGING_REPO` is set to `github.event.pull_request.head.repo.clone_url` (the fork's repo URL when the PR comes from a fork).
4. Runs `percona-obs -P pr-<N> sync push --branch-from main`:
   - For each package, compares `git log <main-last-sync-sha>..HEAD -- <package-path>`. If no commits → uploads an `_aggregate` file pointing at the corresponding main OBS package (binary reuse, no build needed). If changed → uploads full sources to the PR project.
5. Posts (or updates) a PR comment with the OBS project URL and a table showing how many packages were built from source vs. aggregated from main. The comment is identified by an HTML marker `<!-- obs-pr-check -->` so it is updated in place on subsequent pushes to the PR. If the sync step failed, the comment is updated with an error notice instead.

**Required repository config**: `OBS_APIURL`, `OBS_WEB_URL`, `OBS_ROOTPRJ`, `OBS_PR_ROOTPRJ`, `OBS_USER` (vars); `OBS_PASSWORD` (secret).
Permissions: `contents: read`, `pull-requests: write`.

`OBS_PR_ROOTPRJ` is the base prefix for PR-specific projects, e.g. `home:Admin:percona:pr`. The full PR project becomes `home:Admin:percona:pr:pr-42`. It is intentionally separate from `OBS_ROOTPRJ` so PR projects live in a distinct namespace and are never confused with the main project tree.

### Workflow 3 — `obs-pr-cleanup.yml` (release or delete PR project on close)

**Trigger**: `pull_request` against `main` (type: `closed`) where at least one file under `root/**` changed.

**What it does**: Two paths depending on whether the PR is a release PR (i.e. it created a version tag).

**Release PR path** (a version tag like `ppg/17.9.1` was pushed during the PR lifecycle):
The PR built packages against a specific tagged upstream version and those binaries are already in the
production project via the normal release pipeline. Only cleanup is needed:
- Runs `percona-obs -A $OBS_APIURL -R $OBS_PR_ROOTPRJ:pr-<N> sync delete --yes --recursive`.

**Non-release PR path** (no version tag — only packaging or config changes):
The PR's OBS builds contain binaries that need to be promoted to the production project before cleanup:
1. Runs `percona-obs -A $OBS_APIURL -R $OBS_PR_ROOTPRJ:pr-<N> sync release-pr --yes` to copy
   binaries from every PR sub-project into the corresponding production project (via `osc release`)
   and to sync any updated project configs (e.g. newly added architectures) to the production OBS.
2. Runs `percona-obs -A $OBS_APIURL -R $OBS_PR_ROOTPRJ:pr-<N> sync delete --yes --recursive`
   to delete the PR project and all its sub-projects.

OBS 404 responses are handled gracefully in `sync delete` so the cleanup is safe even if the PR
check never ran. `--recursive` ensures projects are deleted even if a build was still in progress.

**Required repository config**: `OBS_APIURL`, `OBS_PR_ROOTPRJ`, `OBS_USER` (vars); `OBS_PASSWORD` (secret).

### Workflow 4 — `obs-stale-cleanup.yml` (daily cleanup of stale PR projects)

**Trigger**: scheduled daily at 02:00 UTC, or manually via `workflow_dispatch`.

**What it does**: Lists all open PRs that have had no activity for more than 7 days and runs `percona-obs sync delete --yes --recursive` against each PR's OBS project (`OBS_PR_ROOTPRJ:pr-<N>`). This prevents inactive PR projects from accumulating indefinitely on OBS. The delete is a no-op if the project never existed (e.g. the PR never touched `root/`).

**Required repository config**: `OBS_APIURL`, `OBS_PR_ROOTPRJ`, `OBS_USER` (vars); `OBS_PASSWORD` (secret).

### Service file env vars

`${VAR}` tokens in `obs/_service`, `obs/_aggregate`, and `obs/_link` files are substituted by `apply_env_substitution()` before the file is uploaded to OBS. The following variables are available:

| Variable | Source | Purpose |
|---|---|---|
| `PERCONA_OBS_PACKAGING_BRANCH` | Profile env | Git ref OBS checks out from the packaging repo (e.g. `main`, `refs/pull/42/head`) |
| `PERCONA_OBS_PACKAGING_REPO` | Profile env | HTTPS clone URL of the packaging repo OBS fetches from. Set to the fork's URL for PR profiles so OBS fetches packaging files from the correct repo when building promoted packages. |
| `REMOTE_OBS_ORG_INTERCONNECT` | Profile env | Prefix for external OBS instance project references (e.g. `openSUSE.org:`). Empty string when no interconnect is used. |
| `OBS_ROOTPRJ` | Auto-injected | The root OBS project name (`--rootprj`). Use this in `_aggregate` files to reference sibling subprojects without hardcoding the org prefix (e.g. `${OBS_ROOTPRJ}:common:deps:runtime`). |
| `DEBIAN_PACKAGE_DIRECTORY` | Auto-injected per package | Path to the package's `debian/` subdir relative to the repo root (e.g. `root/ppg/staging/17/percona-haproxy/debian`). Use as the `subdir` param in the first packaging `obs_scm` service. |
| `RPM_PACKAGE_DIRECTORY` | Auto-injected per package | Path to the package's `rpm/` subdir relative to the repo root (e.g. `root/ppg/staging/17/percona-haproxy/rpm`). Use as the `subdir` param in the second packaging `obs_scm` service. |

`PERCONA_OBS_PACKAGING_BRANCH`, `PERCONA_OBS_PACKAGING_REPO`, and `REMOTE_OBS_ORG_INTERCONNECT` are declared in each `percona-obs` profile via `-e KEY:VALUE` at profile-creation time. `OBS_ROOTPRJ`, `DEBIAN_PACKAGE_DIRECTORY`, and `RPM_PACKAGE_DIRECTORY` are injected automatically and do not need to be declared manually.

---
> Source: [percona/obs-packaging](https://github.com/percona/obs-packaging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
