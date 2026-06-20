## codemap

> Generate a codetree (comprehensive codebase map as nested markdown) and a codemap.json (visual branch terrain map) for any project. Use when the user asks to map a codebase, generate a codemap, create a codetree, understand a project's branch landscape, or produce a visual overview of a repository's architecture and work-in-progress.


# Codemap Skill

Produce two artifacts for a repository:

1. **`<project>-codetree.md`** — a deep nested-list document covering every file, system, branch, and possible task in the codebase
2. **`<project>-codemap.json`** — a terrain map JSON consumed by the Codemap viewer, placing branches as landmarks across a procedurally-generated world

Both files are written to the project's root directory.

---

## Phase 1 — Explore the Codebase

Explore thoroughly before writing anything. The codetree's quality depends entirely on how well the codebase is understood.

### What to gather

**Repository structure**
```bash
ls <project-root>/
# Read README.md, package.json / pyproject.toml / Cargo.toml / go.mod
# Read the main config files: vite.config.*, next.config.*, vercel.json, etc.
```

**Source files** — read every meaningful file in `src/`, `lib/`, `app/`, `api/`, `backend/`, etc. For large files (>500 lines), read enough to understand:
- What the file does
- Its key exports, functions, or components
- Dependencies on other files
- Architectural patterns it establishes

**Git branches and history**
```bash
git -C <project-root> branch -a
git -C <project-root> remote get-url origin
git -C <project-root> symbolic-ref refs/remotes/origin/HEAD
git -C <project-root> log --oneline --all --graph -80

# Full dated commit log for timeline reconstruction
git -C <project-root> log --format="%h %ad %s" --date=short -120

# If the log is long, page through it
git -C <project-root> log --format="%h %ad %s" --date=short -120 --skip=120

# PR merges only (useful for epoch boundaries)
git -C <project-root> log --oneline --merges
```

**What to extract from the history:**
- First commit date and message (the "epoch 0" anchor)
- HEAD commit hash, date, and message
- Origin remote URL and default branch, if present. If there is no remote, omit `repo`.
- Total commit count on main: `git rev-list --count HEAD`
- List of merge commits (identifies PRs and branch landings)
- Any large direct-to-main bursts (multiple commits same day, no merge parent)
- Gaps between activity clusters (quiet periods between epochs)
- Author names from commit log

**Related projects** — if sibling directories look related (e.g. `project-ios`, `project-map`), do a quick `ls` and README read.

### What to capture per file
- Purpose in one sentence
- Key functions / components / exports
- Notable patterns or technical decisions
- Dependencies on other files

---

## Phase 2 — Write the Codetree

Write `<project>-codetree.md` in the project root. The codetree is a comprehensive nested markdown document. Use `##` for top-level sections and nested `-` lists for everything else.

### Required sections

#### 1. Project Overview & Architecture
- What the product does (one paragraph)
- High-level data flow (request → processing → response)
- Interaction modes or user flows
- Repository layout (top-level dirs and what they contain)

#### 2. Frontend — `src/` (or equivalent)
One subsection per significant file or component group. For each:
- What it does
- Key sub-topics a developer might ask about (e.g. "Explain the generation state machine")
- Refactor targets if the file is large (note with "**Refactor targets (see §6):**")
- Delete candidates if stale code exists (note with "**Delete candidates (see §7):**")

#### 3. Backend / API — `api/` or `backend/` (or equivalent)
Same treatment as Frontend. Break into subsections per route file, handler, or service. For large library files, enumerate their major sections:
- Configuration
- Data models / error types
- Core logic functions (list each with a one-line description)
- Utilities

#### 4. Configuration & Deployment
- Build tooling (Vite, webpack, etc.) — multi-entry points, output config
- Deployment config (Vercel, Netlify, Dockerfile, etc.)
- Environment variables — group by purpose (model keys, infra, runtime, feature flags)

#### 5. Active Branches & Directions of Work
One subsection per branch (or logical group of related branches). For each:
- Branch name(s)
- What the work is (description from recent commit messages)
- Direction — what it's trying to achieve
- Status — in progress / merged / stale / experimental

Group related branches together: e.g. all `username/scaling-*` variants under one heading.

#### 6. Refactoring Opportunities
Specific, actionable extraction targets. Name the current file, the proposed new file/module, and what moves there. Format:
```
### 6.1 `src/App.jsx` decomposition (~N lines)
- **Extract WaitRoom** → `src/components/WaitRoom.jsx`
- **Extract generation SSE logic** → `src/lib/useGeneration.js`
```

#### 7. Deletion Candidates
Things that appear safe to remove: legacy keys, dead endpoints, unused model paths, stale branches. Note why each is safe to delete.

#### 8. Features That Could Be Requested
Group by horizon:
- **In-progress / near-term** — features visible in branches, almost ready
- **Product surface expansions** — natural next steps from existing patterns
- **Model / generation improvements** — if AI/ML is involved
- **Backend / infrastructure improvements**
- **Developer / operator features**

#### 9. Related Projects
Any sibling repos, native apps, or companion tools. Brief description and status.

#### 10. Testing & Observability
Current state of tests and monitoring. List what's missing and what would be valuable to add.

#### 11. Repository History
This section turns the raw git log into a narrative that a new contributor can read in 5 minutes. It answers: *What has been built? In what order? What was the hardest sprint? What changed most recently?*

Structure it as:

**§11.1 Velocity overview** — one-line facts:
- First commit date + hash + message
- HEAD date + hash + message
- Total commit count on main
- Number of merged PRs
- Contributors (from git log `--format="%an"`)
- Currently active branch(es)

**§11.2 Milestone timeline** — one subsection per epoch. An epoch is a natural cluster of related work that shipped together. Identify epochs by:
- Quiet gaps between bursts of commits
- "Merge pull request" commit clusters
- A clear theme shift (e.g., "everything in this week is about the wait room")

For each epoch include:
- Date range and duration
- A table of the most important commits (hash, change description)
- **State at epoch end** — what was true about the product/system when this epoch finished

**§11.3 Notable direct-to-main commits** — a table of commits that bypassed branch review. Columns: date, hash, change, why-direct (urgency fix, solo addition, breaking API change, etc.)

**§11.4 Full PR log** — a table: PR number | branch | merge date | one-line description. Sorted chronologically.

---

## Phase 3 — Write the Codemap JSON

Write `<project>-codemap.json` to the project root as well, as a sibling to the codetree file.

The JSON is consumed by the Codemap viewer app. Every branch in the repo becomes a landmark on a procedurally-generated terrain map. Semantically-related branches should cluster in the same region; the biome signals the nature of the work.

### Updating an existing codemap

Before writing a codemap, check whether `<project>-codemap.json` already exists. If it does, treat the existing file as the stable map geography.

Default update behavior:
- Preserve existing `position` values for entries whose names still exist.
- Preserve existing `region`, region labels, and biomes unless the existing assignment is clearly wrong or the user asks for a layout refresh.
- Update metadata in place: `head`, `repo`, `kind`, `status`, `ahead`, `behind`, `lastCommit`, `message`, `pr`, `reviewers`, and `commits`.
- Add new real branches near the existing cluster that best matches their touched files, subsystem, or region.
- Add new suggested work near the existing region it would affect; create a new region only when it represents a genuinely new product/system area.
- Add new milestone and hotpatch entries near the topical cluster they summarize, slightly closer to main than active work when appropriate.
- Remove or mark stale entries only when the underlying branch/work no longer exists or the current analysis shows it is obsolete.
- Keep `seed` stable so terrain noise does not change between updates.

Do not recompute the whole layout just because branch counts, commit counts, or suggested work changed. Moving existing landmarks makes the map harder to read over time. Only move existing entries when positions are invalid, duplicated, grossly misleading, or the user explicitly asks to reorganize the map.

### Schema

```jsonc
{
  "$schema": "codemap@1",
  "name": "owner/repo",          // repo name for display
  "repo": {
    "remoteUrl": "git@github.com:owner/repo.git", // from `git remote get-url origin`, if present
    "webUrl": "https://github.com/owner/repo",    // normalized GitHub web URL, if derivable
    "defaultBranch": "main"                       // branch used as the PR compare base
  },
  "seed": 1337,                  // integer — controls terrain noise; pick any number
  "head": "main",               // currently checked-out branch
  "regions": {
    "<region-id>": {
      "label": "Human Label",   // shown in UI filters and popups
      "biome": "<biome-key>"    // see Biomes below — determines terrain appearance
    }
  },
  "branches": [
    {
      "name": "feature/foo",    // branch name (or conceptual name for planned work)
      "kind": "<kind>",         // see Kinds below
      "region": "<region-id>",  // must match a key in regions — determines biome/terrain
      "position": [-1.5, -2.6], // [x, y] origin-centered — main at [0,0], unbounded — where this branch sits on the map
      "icon": "<icon-key>",     // see Icons below
      "author": "username",     // git author or "—" for planned
      "commits": 12,            // commit count (0 for planned)
      "status": "<status>",     // see Statuses below; omit for kind: "suggested"
      "ahead": 5,               // commits ahead of main
      "behind": 3,              // commits behind main
      "lastCommit": "2d ago",   // relative time string or "—"
      "message": "Description of what this branch/item does",
      "pr": "#123",             // PR number string or null
      "reviewers": ["alice"]    // array of reviewer usernames
    }
  ]
}
```

**Important:** The `position` field is the primary layout mechanism. Coordinates are origin-centered: main sits at `[0, 0]` and other branches spread outward with no fixed bounds. The renderer builds terrain (landmasses, coastlines, water) directly around branch positions. The hex grid is sized dynamically from the data. Regions only control biome appearance (what the terrain looks like), not where things go.

### Biomes

Choose the biome that matches the nature of the work in each region:

| Biome | Use for |
|-------|---------|
| `plains` | Mainline, stable, merged production code |
| `forest` | Active feature development, growing systems |
| `mountain` | Infrastructure, scaling, hard problems |
| `swamp` | Legacy, cleanup, technical debt, stale branches |
| `desert` | Analytics, observability, config, arid utilities |
| `volcanic` | Performance-critical, high-urgency active work |

### Icons

| Icon | Use for |
|------|---------|
| `castle` | Main branch, primary trunk |
| `fortress` | Develop / staging protected branches |
| `tower` | Release branches |
| `keep` | Long-lived protected branches |
| `fort` | Infrastructure / security branches |
| `house` | Active feature branches (primary work) |
| `hut` | Smaller feature branches, fixes |
| `tent` | Experimental, in-progress, planned work |
| `pickaxe` | Refactoring, tooling, build changes |
| `rock` | Stale or abandoned branches |
| `ruin` | Legacy branches, frozen code |
| `ship` | Streaming, real-time, transport layer work |
| `obelisk` | Design system, tokens, visual identity |
| `volcano` | Performance-critical, high-urgency active work |

### Kinds

| Kind | Meaning |
|------|---------|
| `branch` | real git branch, protected branch, or release branch |
| `milestone` | historical development epoch |
| `hotpatch` | notable direct-to-main commit cluster |
| `suggested` | codemap-generated planned/proposed work that does not exist yet |

### Statuses

| Status | Meaning | Color |
|--------|---------|-------|
| `protected` | main, staging — cannot be deleted | gold |
| `release` | release/* branches | purple |
| `open` | active open PR or in-progress local work | blue |
| `draft` | draft PR (exists in git, not yet ready for review) | tan |
| `merged` | merged into main, still listed for history | green |
| `stale` | not updated in weeks, likely abandoned | grey |

### Designing branch positions

Position is the most important field in the codemap. It encodes **magnitude and direction of change** relative to main and to other branches. The renderer builds terrain around whatever positions you specify — you have full control. The world grid is sized dynamically from the bounding box of all positions, so there is no hard boundary.

**Core principles:**

1. **Main at origin.** Place `main` (or equivalent trunk) at `[0, 0]`.

2. **Distance from main = divergence magnitude.** Branches with small changes (1-5 commits, nearly merged) should be close to [0,0]. Branches with large divergence (50+ commits ahead, experimental rewrites) should be far away. Use the `ahead` count as a primary signal.

3. **Direction from main = area of change.** Branches that modify the same subsystem or files should radiate in the same direction. For example:
   - Auth/security work → upper-left (negative x, negative y)
   - Frontend/UI work → north (negative y)
   - Backend/infra work → right (positive x)
   - Data/storage → lower-right (positive x, positive y)
   - Docs/config → near origin (small changes)

4. **Branches with file overlap cluster together.** If two branches touch the same files, they should be near each other regardless of region. This is more important than region grouping.

5. **Clusters form landmasses.** The renderer builds land under every branch position (radius ~10 hexes at HEX_DENSITY=10). Branches within ~1.0 unit of each other merge into a single landmass. Use this to control continent shapes — tight clusters become islands, nearby clusters merge into continents.

6. **Expand outward for new areas.** Rather than rearranging existing positions when a codebase grows, expand outward into new territory. The world grid grows dynamically.

**Position assignment process:**

```
1. Place main at [0, 0]
2. For each branch, compute:
   - divergence = ahead / max_ahead_in_repo (normalized 0–1)
   - angle = based on which subsystem/area of code is being changed
3. Position = divergence * 4.0 in the chosen direction
4. Adjust to avoid exact overlaps (min ~0.2 apart)
5. Verify clusters are separated by at least ~1.5 from unrelated clusters
```

**Typical layout for a full-stack web app:**
```
  [-3.2, -3.0] fuzz        [3.0, -3.0] docs

  [-2.2, -2.4] auth        [2.2, -1.2] query/features

  [-3.2, -0.8] bugs      [0, 0] MAIN        [3.4, 0.6] infra

  [-2.0, 1.7] onboarding              [3.6, 1.7] networking

  [-4.0, 3.8] legacy   [0.6, 3.0] perf   [2.4, 2.8] design
```

**Key constraints:**
- No two branches at the exact same position
- Branches within the same region should be within ~0.5 units of each other
- Unrelated clusters should be at least ~1.5 units apart (creates water between them)
- No hard boundary — positions can extend as needed

### Including repository history as branches

Historical branches — milestones, merged feature branches, and hotpatches — are **extinct branches**. They represent real past states of the code, not metadata about main. They should be positioned **topically**, near the cluster of work they relate to, and visually distinguished as ruins.

#### Milestones — development epochs
Add one branch per epoch. Epochs are natural clusters of thematically related work identified from the commit log. A new contributor should be able to read these 6–10 entries top-to-bottom and understand the full development arc.

- **region:** assign to the region that best matches the epoch's primary topic (e.g. an auth epoch goes in the auth region)
- **positions:** place near the topically relevant cluster, slightly closer to main than active branches in that cluster (they've already been merged — less divergent)
- **kind:** `milestone`
- **status:** `merged` for all
- **icon:** use the icon that matches the type of work (e.g. `house` for features, `pickaxe` for refactoring, `fort` for infra)
- **name:** `milestone/<N>-<slug>` e.g. `milestone/1-foundation`, `milestone/6-waitroom-launch`
- **commits:** total commit count for that epoch
- **behind:** how many commits behind HEAD the epoch's last commit is
- **lastCommit:** the ISO date of the epoch's last commit (`"2026-04-09"`)
- **message:** one sentence: "Epoch N — [theme]: [key things shipped]"
- If an epoch spans multiple topics, assign it to the dominant one. If it's truly cross-cutting (e.g. "initial project setup"), place it near main.

#### Hotpatches — direct-to-main commits
Add one branch per notable cluster of direct-to-main commits (bypassed branch/PR workflow). These are important for new contributors to know: they show what the team considers urgent enough to skip review.

- **region:** assign to the region the hotpatch affected (e.g. a payments hotfix goes in the payments region)
- **positions:** place near the topically relevant cluster, close to main (hotpatches are small divergences by nature)
- **kind:** `hotpatch`
- **status:** `merged` for all
- **icon:** use the icon that matches the type of work (e.g. `volcano` for emergencies, `pickaxe` for fixes)
- **name:** `hotpatch/<slug>` e.g. `hotpatch/admission-cap-crisis`, `hotpatch/gemini-migration`
- **message:** describe what was changed and why it bypassed review (capacity emergency, breaking API change, solo addition, etc.)

How to identify direct-to-main commits from the log:
- A cluster of commits with no merge parents on the same day or short window
- Subject lines like "Hotfix:", "Raise X cap", "Fix X format", "Update X" without a corresponding PR merge above them
- Run `git log --no-merges --format="%h %ad %s" --date=short` and look for same-day clusters

#### Visual treatment of merged branches
All branches with `status: "merged"` are rendered as ruins by the renderer — the terrain tiles around them are desaturated with a gray wash, making them visually distinct from active landmasses at any zoom level. No special icon or data-level treatment is needed beyond setting `status: "merged"`. Use the icon that semantically describes the type of work.

### Including planned work as branches

The codemap is not just a git branch viewer — it is a map of the entire work landscape, including work that hasn't started yet.

For every refactoring opportunity, deletion candidate, and expected feature from the codetree, add a corresponding entry in the codemap with:
- A descriptive `name` that follows existing naming conventions (e.g. `refactor/split-lib`, `feat/graph-view`, `cleanup/legacy-keys`)
- `kind: "suggested"` and no `status`
- `commits: 0`, `ahead: 0`, `behind: 0`
- `author: "—"`, `lastCommit: "—"`
- A `message` that is the one-line summary of the task from the codetree

This makes the codemap a complete picture of the current state AND the future roadmap.

### Branch count and coverage

Aim for 40–70 branches minimum. Include:
- All real local branches
- All significant remote branches (skip trivial one-commit remote branches)
- One entry per major refactoring opportunity from the codetree
- One entry per near-term planned feature from the codetree
- One entry per deletion / cleanup candidate from the codetree

---

## Output checklist

Before finishing, verify:

- [ ] `<project>-codetree.md` written to project root with all 11 sections
- [ ] Every significant source file is mentioned in the codetree with a description
- [ ] All git branches appear in §5 of the codetree
- [ ] All real git entries use `kind: "branch"` and a lifecycle `status`
- [ ] All refactor opportunities in §6 have a corresponding `kind: "suggested"` entry in the codemap
- [ ] All planned features in §8 have a corresponding `kind: "suggested"` entry in the codemap
- [ ] All deletion candidates in §7 have a corresponding `kind: "suggested"` entry in the codemap
- [ ] §11 history section covers: velocity stats, epoch timeline with state-at-end, direct-to-main table, full PR log
- [ ] `<project>-codemap.json` written with valid JSON (no trailing commas)
- [ ] If updating an existing codemap, existing positions and seed are preserved unless a layout refresh was requested
- [ ] If updating an existing codemap, new entries are placed near related existing clusters rather than triggering a full rearrangement
- [ ] Every branch's `region` value matches a key in `regions`
- [ ] Every branch has a valid `kind` value
- [ ] Every branch has a `position: [x, y]` field (origin-centered, unbounded)
- [ ] `main` is positioned at [0, 0]
- [ ] Branch positions reflect divergence: high-ahead branches are farther from origin
- [ ] Branch positions reflect affinity: branches touching similar files are near each other
- [ ] Clusters of related branches are within ~0.5 units of each other
- [ ] Unrelated clusters are at least ~1.5 units apart (creates water between them)
- [ ] No two branches at the exact same position (min ~0.2 apart)
- [ ] Milestones present (one per epoch), positioned near their topically relevant cluster, closer to main than active branches
- [ ] Hotpatches present for notable direct-to-main commit clusters, positioned near the topic they affected
- [ ] All merged/historical branches use `status: "merged"` (renderer handles visual treatment)
- [ ] All merged branch `lastCommit` values are real ISO dates or accurate relative strings — not estimates
- [ ] All merged branch `behind` values are non-zero (they're behind HEAD by definition)
- [ ] `head` matches the actual current branch from `git branch`
- [ ] `seed` is an arbitrary integer (use the project name's char codes or any memorable number)

---
> Source: [tarzain/codemap](https://github.com/tarzain/codemap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
