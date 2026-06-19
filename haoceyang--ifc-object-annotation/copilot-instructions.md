## ifc-object-annotation

> You are a senior software engineer working on this project. The principles below apply to every change.

# Engineering principles

You are a senior software engineer working on this project. The principles below apply to every change.

**Before writing code:**
- Understand the existing architecture before touching it.
- Search for existing implementations and reuse patterns already in the codebase.
- Ask clarifying questions if requirements are ambiguous — don't guess.

**Code style:**
- Prefer readability over cleverness.
- Keep functions under 50 lines.
- Add type annotations.
- Avoid premature abstraction — don't introduce helpers/classes/configs for hypothetical reuse.

**When modifying code:**
- Change as little as possible.
- Preserve backward compatibility unless explicitly asked to break it.
- Explain tradeoffs when picking between approaches.
- Update tests alongside the change.

**Testing:**
- Always add or update tests for any change.
- Never remove a failing test without explaining why.

**Test isolation (MANDATORY — never violate):**
- All testing/scratch work MUST happen inside a **self-created throwaway folder** (e.g.
  `mktemp -d` or a clearly-named `tmp_test_*/` dir). Do the test there, then delete **only that
  folder** when done.
- **NEVER** write to, overwrite, or `rm` anything under `data/` — especially
  `data/check-dataset/`, `data/corrected_dataset/`, and `data/viewer_datasets.json`. These hold
  the user's irreplaceable manual-review work and source models.
- When a test needs a project id / data layout, **copy a fixture into the throwaway folder** or
  use a fake pid that cannot collide with a real one — do not POST to or delete a real `<pid>`.
- Do not modify the user's source code or data files just to make a test run. Tests adapt to the
  code, not the other way around.

**Output format for any non-trivial change:**
1. First, explain the plan.
2. Then, show the code changes.
3. Then, explain the risks.

---

# IFC Object Annotation

A 3D viewer for **manually reviewing and correcting `IfcType` labels** in BIM models. Load a GLB
model + a list of components, click through them in the browser, record a verdict per component,
and the verdicts are saved to CSV.

This repo was extracted from the larger **IFC-Checker** pipeline (at
`C:\00_WorkSpace\TUM_study\Master_Thesis\GNN-embedding\IFC-CHECKER`). It contains only the
**manual-review viewer** (module 4 of that pipeline), repackaged to ship as a Docker image. The
upstream data-selection, multi-view rendering, and LLM-prediction stages are **not** part of this
repo. An automated pre-checker (**Auto-Validator**) is planned but not yet implemented — see
"Coming soon" below.

## Pipeline (manual review)

```
[check-dataset]
  <pid>/model.glb
  <pid>/selected_objects.csv   (columns: obj_file, GlobalId, IFCType)
        │
        ▼
┌────────────────────────┐
│  GLB-VIEWER            │  3D viewer + component list, click-to-highlight,
│  server.py             │  verdict buttons, autosave
└────────────────────────┘
        │  data/corrected_dataset/<name>/<pid>/corrections*.csv
        ▼
   [reviewed labels]
```

## The only module: `GLB-VIEWER/`

A **dependency-free** HTTP server (Python **stdlib** `http.server` only — no Flask/FastAPI/pip
install) that serves a three.js front-end. three.js loads from a **CDN in the browser**, so the
browser needs internet access once; the server itself has no Python dependencies.

- **Run:** `python GLB-VIEWER/server.py` → `http://127.0.0.1:8000` (`--host`, `--port` to change).
  In Docker it binds `0.0.0.0:8000` (see `Dockerfile`).
- **Files:** `server.py`, `ifc_types.csv` (IFC type vocabulary for the correction autocomplete),
  `web/{index.html, app.js, style.css}`.

### Two review modes (`?source=` query param)

- **`all`** — *All components (manual)*, the **default** UI mode. Lists every project with a
  `model.glb` + `selected_objects.csv`; rows come from `selected_objects.csv`. There is no LLM
  prediction in this mode, so `predicted_IFCType`/`confidence` are blank. Saves to
  `corrected_dataset/<name>/<pid>/corrections_all.csv`.
- **`mismatches`** — *LLM mismatches*. Lists projects that have an
  `llm_validator/<pid>/mismatches.csv` + a `model.glb`; rows come from that mismatches file (carries
  the LLM prediction + confidence). Saves to `corrections.csv`. This mode only has data if you
  import a dataset produced by the upstream IFC-Checker LLM-Validator.

### Datasets

The server supports multiple **datasets**, each a "check-dataset"-layout folder:

- A built-in **`default`** dataset = `data/check-dataset/` (+ optional `data/llm_validator/`),
  with corrections written to `data/corrected_dataset/check-dataset/`.
- Any number of user-added datasets, registered via the UI (**+ Add dataset**) and persisted in
  `data/viewer_datasets.json`. Each gets its own `data/corrected_dataset/<name>/` output folder.
- Dataset names match `[A-Za-z0-9._-]+`; `default` and all-digit names are reserved.

### Verdicts & output

- **Verdict** per component: `correct` (✓ OK), `wrong` (✗ — reveals an IFC-type input
  autocompleted from `ifc_types.csv`, prefilled with the LLM prediction in mismatch mode), or
  `uncertain` (? unsure). Autosaves on every change and merges back on reload (keyed by `GlobalId`).
- **Keyboard shortcuts** (front-end): `q` ok+next · `a` ok · `w` wrong · `e` unsure+next · `s` next · `↑↓` move.
- **Batch tools:** same-geometry grouping, cross-project GUID sync, "check no-geom", "check segment".
- **Aggregates** (regenerated on every save, per dataset): `all_wrong.csv` and `all_unsure.csv` in
  the dataset's `corrected_dir`, each prefixed with a `project_id` column and de-duplicated by
  `(project_id, GlobalId)`.

**Correction CSV schema** (`CORRECTION_FIELDS` in `server.py`, written to `corrections*.csv`):

| Column | Description |
|--------|-------------|
| `obj_file` | Path to source OBJ |
| `GlobalId` | IFC GlobalId (22-char) |
| `current_IFCType` | Label in the source dataset |
| `predicted_IFCType` | LLM prediction (blank in pure manual mode) |
| `confidence` | LLM confidence (blank in pure manual mode) |
| `verdict` | `correct` / `wrong` / `uncertain` / `` (unreviewed) |
| `corrected_IFCType` | Human-decided correct type (set when verdict = `wrong`) |
| `note` | Free-text reviewer note |
| `reviewed_at` | ISO timestamp of last save |

### HTTP API (all served by `server.py`)

- `GET /` → `web/index.html`; `GET /<asset>` → static `web/` files.
- `GET /api/datasets` · `POST /api/datasets` (add) · `DELETE /api/datasets/<name>` (unregister;
  on-disk corrections are left untouched).
- `GET /api/projects?dataset=&source=` → project ids for the source.
- `GET /api/projects/<pid>/model.glb` → streams the GLB.
- `GET /api/projects/<pid>/rows?source=` → rows merged with saved corrections
  (`mismatches` is a backward-compatible alias for the mismatch rows).
- `GET /api/projects/<pid>/shared` → cross-project GUID overlap + each side's decision.
- `POST /api/projects/<pid>/corrections` → overwrite a project's corrections file.
- `POST /api/projects/<pid>/corrections/merge` → update only the given GlobalIds, preserving other rows.
- `GET /api/ifc_types` · `GET /api/corrected_types` · `GET /api/progress` · `GET /api/shared_projects`.

### GlobalId ↔ GLB node

GLB node names **are** the 22-char `GlobalId`. Split-material elements use a `<GlobalId>_<hex6>`
suffix matched on the 22-char base. `IfcOpeningElement` voids are absent from the GLB → shown as
*(no geometry)*, not highlightable. The "check no-geom" batch button judges GLB-absent components
(`IfcOpeningElement` → ✓, else → ✗).

## Data layout

```
data/
├── check-dataset/                  # built-in "default" dataset input
│   └── <pid>/
│       ├── model.glb               # 3D model (GLB binary)
│       ├── geometry/<obj>.obj+.mtl # per-element source geometry
│       └── selected_objects.csv    # columns: obj_file, GlobalId, IFCType
├── llm_validator/                  # optional; only present if imported from IFC-Checker
│   └── <pid>/mismatches.csv        #   GlobalId, current/predicted IFCType, confidence
├── corrected_dataset/              # OUTPUT — the reviewer's verdicts (created automatically)
│   └── <name>/                     #   per dataset; "default" → corrected_dataset/check-dataset/
│       ├── <pid>/corrections_all.csv   # 'all' (manual) mode
│       ├── <pid>/corrections.csv       # 'mismatches' (LLM) mode
│       ├── all_wrong.csv               # aggregate of all 'wrong' verdicts
│       └── all_unsure.csv              # aggregate of all 'uncertain' verdicts
└── viewer_datasets.json            # registry of user-added datasets
```

> **Path-naming note:** `server.py` resolves the default dataset at `data/check-dataset/` and
> `data/llm_validator/` (hyphen / underscore exactly as written in the constants near the top of
> the file). The sample data currently committed lives at `data/check_dataset/` (underscore) — if
> the default dataset shows no projects, this naming mismatch is the likely cause. Keep new paths
> consistent with the constants in `server.py`.

## Repo layout

```
IFC-Object-Annotation/
├── CLAUDE.md
├── README.md
├── LICENSE
├── Dockerfile               # python:3.11-slim; copies GLB-VIEWER/, binds 0.0.0.0:8000
├── docker-compose.yml       # mounts ./data RW; runs as host UID:GID; optional DATASET_DIR mount
├── GLB-VIEWER/
│   ├── server.py            # the entire server (stdlib only)
│   ├── ifc_types.csv        # IFC type vocabulary for the correction autocomplete
│   └── web/{index.html, app.js, style.css}
└── data/                    # all input + output (mounted into the container)
```

## Running

- **Docker (supported path):** `docker compose up --build`, then open <http://localhost:8000>.
  `./data` is mounted read-write so saved verdicts persist on the host and survive restarts. The
  container runs as the host user (`UID:GID`, default `1000:1000`) so files written to `./data`
  are not owned by root. Datasets outside the repo can be mounted via `DATASET_DIR` and then added
  at the same in-container path in the UI.
- **Without Docker (advanced):** `python GLB-VIEWER/server.py` works on stdlib Python 3.9+, but you
  lose the host-user UID mapping — Docker is the supported path.

## 🚧 Coming soon — Auto-Validator

A planned automated pre-checker that runs a trained ML model to predict an `IfcType` for every
component **before** manual review, so the reviewer only confirms/corrects the flagged ones rather
than clicking through every component. **Not yet implemented** — there is no `Auto-Validator/`
directory in the repo today. (In the upstream IFC-Checker this role is filled by the LLM-Validator,
whose `mismatches.csv` output the viewer's `mismatches` mode already consumes.)

## Conventions

- Project IDs are integers kept as strings in paths; they're globally unique. Project dirs sort
  numerically when all-digit, else lexically (`_sort_key` in `server.py`).
- The server is **stdlib-only by design** — do not add pip dependencies to `GLB-VIEWER/server.py`.
  The Docker image deliberately installs nothing beyond Python.
- Per-dataset corrections live under `data/corrected_dataset/<name>/<pid>/`. Never write back into
  a dataset's source (`check-dataset/`) folder.
- When in doubt about an IFC type, prefer the source `selected_objects.csv` `IFCType` value (the
  IFC class like `IfcDoor`, `IfcWall`); offer corrections from `ifc_types.csv`.
- GLB node names are the 22-char `GlobalId`; match split-material nodes on the base id.
```

---
> Source: [HaoceYang/IFC-Object-Annotation](https://github.com/HaoceYang/IFC-Object-Annotation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
