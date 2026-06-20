## skill2gep

> Generic skill distillation tool. Converts any procedural Skill document (Cursor skill, Claude/Anthropic skill, or any SKILL.md / workflow-style markdown) into two kinds of GEP (Gene Evolution Protocol) assets. Gene = compact strategy template (signals + strategy + AVOID + validation). Capsule = audit record of a real execution of a Gene (outcome + execution_trace + env_fingerprint). Capsule must be produced from "Gene + at least one real-world execution"; fabricating a Capsule from the document alone is forbidden. Use when the user says "turn this skill into genes", "skill2gep", "skill2gene", "skill2capsule", "distill skill into GEP assets", or asks to compress a SKILL.md / procedural document into GEP control signals.


# skill2gep

A generic tool that is NOT tied to any specific agent platform. Input: any procedural Skill document (written for humans). Output: two kinds of GEP assets.

The essential distinction (per the GEP protocol definition):

| Asset | What it is | Where it comes from | Can be distilled directly from a document? |
|---|---|---|---|
| **Gene** | Reusable strategy template. Defines "under which signals, do what, avoid what, how to validate" | An abstraction of experience. An empty template | Yes |
| **Capsule** | Audit record of one concrete execution (outcome, env_fingerprint, trigger, reference to the Gene used) | Gene + one real execution | **No.** Must start from an existing Gene and run it once to produce a result |

This is a hard constraint. Violating it = fabricating a Capsule = destroying the "from validation" foundation of GEP.

## Core principles (non-negotiable)

1. **From validation only**. An unvalidated Gene must not be installed; a Capsule without a real execution must not exist.
2. **Genes serve control density**. Target 200-300 tokens, hard ceiling 500 tokens. If you cannot fit within 500, you are under-splitting the dimensions, not the Gene's fault.
3. **Capsules serve auditability**. Must reference a concrete Gene `id`, must carry a real `outcome`, must carry an `env_fingerprint`.
4. **Failure experience is distilled into `AVOID:` lines**, never mixed into normal strategy steps ("failure warnings only" is the strongest form per Wang et al., *From Procedural Skills to Strategy Genes*, arXiv:2604.15097).
5. **No hallucination**. Every `signals_match`, `strategy` step, and `AVOID` line must trace back to a specific passage in the source document or a real failure record.

## Workflow overview

```
Source Skill document
  |
  +-- Phase 1: Read and analyze the source
  +-- Phase 2: Recall / deduplicate (local + community)
  +-- Phase 3: Split by dimension, draft candidate Genes (strategy only)
  +-- Phase 4: Gene local validation (hard gate: schema + dry-run + scenario replay)
  +-- Phase 5: Install into local Gene pool + record_outcome
  |
  +-- [Path A] Gene only: stop here
  |
  +-- [Path B] Capsule needed (executable path + audit record):
  |     +-- Phase 6: Pick a real scenario + inject an accepted Gene + execute for real
  |     +-- Phase 7: Capture outcome / blast_radius / env_fingerprint / diff
  |     +-- Phase 8: Assemble the Capsule (type=Capsule, gene=<id>, outcome=...)
  |     +-- Phase 9: Validate the Capsule locally -> write to local Capsule pool
  |
  +-- [Optional] Phase 10: Publish to EvoMap community via evolver / EvoMap API
```

By default, produce both Gene and Capsule. Skip one path only when the user explicitly asks for just one kind of asset.

---

## Phase 1: Read and analyze the source Skill

Use any file-reading tool to load the Skill document the user points to. Common locations:

- Cursor skill: `~/.cursor/skills/<name>/SKILL.md` or `.cursor/skills/<name>/SKILL.md`
- Claude / Anthropic skill: project-root `SKILL.md` or `skills/<name>/SKILL.md`
- Any procedural document: an explicit path given by the user (a README "Workflow" section counts)

Extract the following material (**each item must trace back to a source line**, otherwise it is hallucination):

| Material | Which section of the document it comes from | Which Gene field it feeds |
|---|---|---|
| Trigger scenarios, keywords, error signals | Frontmatter `description`, body "Use when / Scenario / Trigger" | `signals_match` |
| Ordered executable steps | Workflow / Quick Start / flow diagram | `strategy` |
| Explicit pitfalls, anti-patterns, "do not ..." | Pitfalls / Common Mistakes / AVOID / Anti-Patterns | `strategy` entries starting with `AVOID:` |
| Modification scope, forbidden areas | Scope / path constraints / "do not touch ..." | `constraints.max_files`, `constraints.forbidden_paths` |
| Executable validation | Validation / Feedback loop / "run xxx to confirm" | `validation` array |
| Real cases | Examples / Case Study / user-provided history | Used in Phase 4.3 scenario replay and Phase 6 execution |

**If the source has no validation section**: Phase 4 will block. Go back to the user to nail down "exactly what signal proves this Gene worked". Do not paper over it with "manual check" or "looks reasonable".

**If the source has no Examples at all**: Path A (Gene only) can still proceed; **Path B (Capsule) must stop** and inform the user "no real scenario available, cannot produce Capsule".

---

## Phase 2: Recall / deduplicate (local + community)

Before drafting candidates, query three channels to avoid duplication. Using the `evomap-gep` MCP as the reference implementation:

```
MCP: evomap-gep / gep_list_genes        -> local gene pool
MCP: evomap-gep / gep_recall            -> historical experience
MCP: evomap-gep / gep_search_community  -> community assets (both Gene and Capsule)
```

If the user runs a different GEP implementation, tool names may differ, but all three semantics must be covered: **local pool + historical events + community pool**.

Match policy:

- **Exact overlap** (a subset of `signals_match` plus the same strategy backbone): stop this distillation and reply "`<id>` already exists, reuse it".
- **Partial overlap**: distill incrementally, only add new signals/strategies, and reference the existing one via `parent_gene: <asset_id>`.
- **No overlap**: proceed to Phase 3.

---

## Phase 3: Split by dimension, draft candidate Genes

A Skill is almost always a mixture of dimensions (happy path + validation + anti-patterns + scope). The point of splitting is to make each Gene's `signals_match` orthogonal so candidates do not steal each other's triggers.

**Common splits** (let the source decide; do not force-fit):

| Dimension | Gene `category` | Typical `signals_match` |
|---|---|---|
| Happy path | `optimize` or `innovate` | task keywords + `normal`, `first_run` |
| Error / failure path | `repair` | `log_error`, specific error keywords |
| Anti-patterns (AVOID-heavy) | `repair` | `tool_bypass`, specific bad-pattern keywords |
| Performance / budget | `optimize` | `perf_bottleneck`, `timeout` |
| Release / deployment wrap-up | `optimize` | `deployment_issue`, `release`, `tag` |

**Required fields per candidate Gene** (`asset_id` and `schema_version` are filled in automatically at install time):

```json
{
  "type": "Gene",
  "id": "gene_<domain>_<intent>_<short_noun>",
  "category": "repair | optimize | innovate",
  "signals_match": ["..."],
  "preconditions": ["..."],
  "strategy": [
    "Step 1: ...",
    "Step 2: ...",
    "AVOID: <specific wrong action> - must first <corrective action>",
    "AVOID: ..."
  ],
  "constraints": {
    "max_files": 12,
    "forbidden_paths": [".git", "node_modules"]
  },
  "validation": [
    "<an agent-runnable command returning 0 / non-zero>"
  ]
}
```

**Writing constraints (hard)**:

- Every `strategy` / `AVOID` line: one imperative sentence, one action. Do not explain "why" (put rationale in `preconditions`).
- `signals_match`: lowercase snake_case. Reuse well-known signals (`log_error`, `perf_bottleneck`, `user_feature_request`, `capability_gap`, `deployment_issue`, `test_failure`, `tool_bypass`) and add domain-specific ones on top.
- `validation`: every entry must be "runnable and gives a clear pass/fail". Never write "check whether the output looks reasonable". No-ops like `echo ok`, `true`, `: ` are rejected by the validator.
- Estimate tokens per Gene (~= 0.5 token per character). Anything over 500 must be split further.

Keep all candidates in memory first. **Do not write to disk yet.**

---

## Phase 4: Gene local validation (hard gate)

Three layers. Failing any one forbids Phase 5.

### 4.1 Schema check

Run `scripts/validate_gene.js <candidates.json>`. It checks:

- All required fields present, `type == "Gene"`, `id` matches `^gene_[a-z0-9_]+$`, no duplicate ids
- `category` is one of the enum, `signals_match` non-empty and all snake_case
- `strategy` has at least one entry, and at least one `AVOID:` entry (if the source genuinely has no failure experience, the agent must explicitly set the waiver flag)
- `constraints.max_files` is a positive integer, `forbidden_paths` is an array
- Each `validation` entry looks runnable (a command, not a human instruction; no-ops are rejected)
- Estimated tokens ≤ 500
- No private path literals leaked (`/home/<user>`, private repo names, etc.)

### 4.2 Dry-run on declared validation commands

The script dry-runs each `validation` entry:

- `node scripts/xxx.js` → `node --check` syntax validation + file existence
- `npx vitest run <path>` → path existence
- `python -m pytest <path>` → path existence
- anything else → `bash -n -c` syntax validation

Goal: block fake Genes whose `validation` is just a placeholder string.

### 4.3 Scenario replay (semantic validation, done by the agent itself)

This step is the soul of "from validation":

1. Pick 1-2 real scenarios from the source Skill's `## Examples` section or from user-provided historical tasks.
2. For each candidate Gene:
   - Check that at least one `signals_match` entry matches the scenario keywords; otherwise this Gene is irrelevant for this scenario.
   - Read the `strategy`: if followed literally, does it solve the example? Are any steps contradicted by the example? Are all pitfalls from the example covered by AVOID lines?
3. Mark Genes that fail as `rejected` or `needs_rewrite`.

### 4.4 Produce a validation report

```
gene_id                                  | schema | dry-run | replay         | decision
gene_uvvis_peak_unit_convert             | pass   | pass    | pass           | accept
gene_uvvis_find_peaks_misuse             | pass   | pass    | pass           | accept
gene_uvvis_overview_blah                 | pass   | pass    | FAIL(no hit)   | reject
```

Only Genes marked `accept` proceed to Phase 5.

---

## Phase 5: Install into the local Gene pool

### 5.1 Recommended: install through MCP / GEP runtime

```
MCP: evomap-gep / gep_install_gene
{ "gene": { ... full Gene JSON ... } }
```

The runtime fills in `asset_id` (SHA-256 content addressing) and `schema_version`.

### 5.2 Fallback: write the asset file directly (when MCP is unavailable)

Append to the GEP runtime's `assets/gep/genes.jsonl` (one JSON object per line). **Do not modify `genes.json` directly** (it is seed data). The exact path depends on the GEP implementation; typically:

```
<evolver or GEP runtime>/assets/gep/
```

After writing, load the file back with the runtime's loader to confirm it parses.

### 5.3 Record outcome

For every Gene successfully persisted, call:

```
MCP: evomap-gep / gep_record_outcome
{
  "geneId": "<new gene id>",
  "signals": ["skill_distillation", <domain signals from the source>],
  "status": "success",
  "score": 0.85,
  "summary": "Distilled <skill-name> into Gene <gene-id> by <splitting strategy>; validated via <brief validation summary>"
}
```

This makes the distillation itself part of the memory graph so it can be recalled later.

**Path A ends here.** Stop if the user only asked for Genes.

---

## Path B: Produce a Capsule (executable path + audit record)

Prerequisite: Phase 5 produced at least one accepted Gene, and the source has a real executable scenario (or the user supplies one).

**Iron rule for Capsules**: a Capsule is not distilled from the document. It is **collected** from the execution of "injecting a Gene into a real task and running it once". If no execution happened, there is no Capsule. Fabricating a Capsule (writing `outcome: success` off the top of your head) destroys GEP's trust foundation.

### Phase 6: Pick a Gene + pick a real scenario + run for real

1. **Pick a Gene**: from the Phase 5 accepted list, pick the one whose `signals_match` overlaps most with the target scenario.
2. **Pick a scenario**: prefer a concrete case from the source Skill's `## Examples` section; otherwise ask the user for a real historical task. **Invented tasks are not accepted.**
3. **Run for real**: execute the Gene's `strategy` step by step in an agent-controlled workspace. This is essentially what the GEP runtime does — if evolver (or an equivalent runtime) is available, let it run; otherwise the agent may run by hand, but must:
   - Record the actual command sequence (as `execution_trace`)
   - Run every command in the `validation` array and record each one's exit code and an output tail
   - Collect the list of modified files and the total changed line count (as `blast_radius`)
   - Collect an environment fingerprint (OS / Node or Python version / key dependency versions, as `env_fingerprint`)

**Absolutely forbidden**: skipping real execution and fabricating an outcome.

### Phase 7: Determine the outcome

Fill the following fields from the Phase 6 measurements:

| Field | How to fill |
|---|---|
| `outcome.status` | All `validation` passed → `success`; any failure → `failed` |
| `outcome.score` | On `success`, score 0.6 - 1.0 based on pass rate and quality; on `failed`, 0.0 - 0.4 |
| `blast_radius.files` | Actual number of modified files |
| `blast_radius.lines` | Total added + removed lines |
| `success_reason` | On `success`, one sentence explaining why the Gene worked here (≤ 200 chars) |
| `env_fingerprint` | `{ "os": "...", "node": "...", "python": "...", "key_deps": {...} }` |

**Failed Capsules are valuable too**: they are stored as `failed_capsule` and become input samples for future `repair` Genes. Do not throw them away just because they failed.

### Phase 8: Assemble the Capsule

Minimum required fields (aligned with the capsule-generation logic in evolver's `solidify.js`):

```json
{
  "type": "Capsule",
  "id": "cap_<timestamp>_<short_hash>",
  "gene": "<accepted gene id from Phase 5>",
  "trigger": ["<signals that actually fired in this execution>"],
  "summary": "<one sentence: real result of applying Gene X to scenario Y>",
  "confidence": 0.0,
  "blast_radius": { "files": 0, "lines": 0 },
  "outcome": { "status": "success | failed", "score": 0.0 },
  "success_reason": "<required on success>",
  "env_fingerprint": { "...": "..." },
  "source_type": "skill2gep_distillation",
  "strategy": [ "<copy of the Gene's strategy array>" ],
  "content": "<structured summary of the Phase 6 run, JSON or plain text>",
  "execution_trace": [
    { "step": 1, "cmd": "...", "exit": 0, "stdout_tail": "..." }
  ]
}
```

**Do not compute `asset_id` yourself**. Let the runtime (`gep_install_*` tools or `assetStore.computeAssetId`) compute it; field-ordering differences would otherwise make the hash unstable.

### Phase 9: Capsule local validation + persist

#### 9.1 Capsule schema check

Run `scripts/validate_capsule.js <capsules.json>`. It checks:

- All required fields present, `type == "Capsule"`
- `gene` refers to an id that actually exists in the local Gene pool (dangling references are forbidden)
- `outcome.status` ∈ {success, failed}; `outcome.score` ∈ [0, 1]
- `blast_radius.files` and `blast_radius.lines` are non-negative integers
- `env_fingerprint` is non-empty
- On the `success` branch, `success_reason` is non-empty
- At least one of `content` / `execution_trace` is non-empty (empty Capsules are forbidden)
- No private path literals leaked

#### 9.2 Cross-reference check

The Capsule's `gene` must resolve inside the local Gene pool. If the Gene was just installed in Phase 5, confirm the install returned success and it shows up in `gep_list_genes`.

**Stricter rule (validation coverage)**: every `cmd` in the Capsule's `execution_trace` must **cover every entry of the referenced Gene's `validation` array** (exact match after whitespace normalization), and the corresponding trace entry must carry a real integer `exit`. This ensures:

- The Gene's `validation` is not decorative — you must actually run it to generate a Capsule
- Faking it requires matching the `execution_trace` to the validation strings exactly, and any forged trace will be exposed by community dedup in Phase 10 or by later real reproductions

When you pass the Gene pool to `validate_capsule.js` via `--genes-jsonl` / `--genes`, this check runs automatically. If no Gene pool is passed, the check is skipped — but the credibility of Phase 9 drops sharply and this is strongly discouraged.

#### 9.3 Persist

Prefer the runtime install channel (e.g. a `gep_install_capsule` tool if provided by the GEP runtime). Fallback: append one JSON object per line to `assets/gep/capsules.jsonl`.

After persisting, call `gep_record_outcome` once more:

```
{
  "geneId": "<the injected gene id>",
  "signals": ["skill2gep_capsule", <scenario signals>],
  "status": "<success | failed>",
  "score": <measured score>,
  "summary": "Capsule <cap_id> from real execution of Gene <gene_id> on <scenario>; outcome=<status>"
}
```

**Path B ends here.**

### Common pitfalls in Path B

- **Skipping real execution**: the most common error. Tell-tale signs: `execution_trace` is empty, `blast_radius.files = 0`, yet `outcome.status = success`. Reject such Capsules.
- **Dangling Gene id**: starting Capsule work before Phase 5 installs finished. Fix: strict sequencing; do not enter Phase 6 before 5 completes.
- **Splitting one execution into multiple Capsules**: one real execution → one Capsule. To get multiple Capsules, run multiple times.
- **Lazy `env_fingerprint` like `"runtime": "unknown"`**: equivalent to nullifying reproducibility. At minimum include OS + runtime major version.

---

## Phase 10 (optional): Publish to the EvoMap public repository

**Only run this phase when the user explicitly says "publish" / "push to EvoMap" / "upload to community".** By default stop at Phase 5 (Path A) or Phase 9 (Path B).

EvoMap exposes two distinct surfaces for publishing distilled work. Pick the right one:

| Surface | What to publish | How |
|---|---|---|
| **Skill Store** (user-facing, downloadable) | A self-contained SKILL.md that another agent can download and run end-to-end | `POST https://evomap.ai/a2a/skill/store/publish` (see [Skill Store wiki](https://evomap.ai/wiki/31-skill-store)) |
| **GEP Hub** (agent-facing evolution memory) | Individual Genes and Capsules (atomic strategy / execution records) | Via the `evolver` runtime (GEP protocol `POST /a2a/memory/record` and the `gep_install_gene` / `gep_search_community` MCP tools) |

### 10.1 Community dedup (before publishing)

Use the GEP MCP tool `gep_search_community` (provided by the `@evomap/gep-mcp-server` reference implementation):

```
MCP: evomap-gep / gep_search_community
{ "query": "<concatenated core signals>", "type": "Gene|Capsule", "limit": 20 }
```

If an equivalent asset is already published, stop and tell the user "community already has this; use `gep_install_gene` to reuse it instead of re-publishing".

### 10.2 Publishing a Skill (recommended path for full SKILL.md files)

The Skill Store is an HTTP API that any registered EvoMap agent can call with `sender_id` + `node_secret`:

```bash
curl -X POST https://evomap.ai/a2a/skill/store/publish \
  -H "Content-Type: application/json" \
  -H "X-Node-Secret: $EVOMAP_NODE_SECRET" \
  --data @skill_publish.json
```

Payload shape (per [Skill Store wiki](https://evomap.ai/wiki/31-skill-store)):

```json
{
  "sender_id": "node_<your_agent_id>",
  "skill_id": "skill_<unique_slug>",
  "content": "---\nname: ...\ndescription: ...\n---\n\n# ...",
  "category": "repair | optimize | innovate",
  "tags": ["..."],
  "bundled_files": [
    { "name": "validate_gene.js", "content": "..." }
  ]
}
```

The Skill goes through 4 layers of security moderation (regex, obfuscation detection, political filter, AI classification) before becoming public. Anti-fragmentation rules also apply: minimum 500 characters, max 5 new skills per author per 24 hours, max 3 skills with the same name prefix per author, >= 85% similarity to an existing skill by the same author is rejected.

### 10.3 Publishing Genes / Capsules (evolver runtime)

GEP assets (Genes, Capsules) live on a separate surface, reached via the evolver runtime rather than a direct REST call:

```bash
# Install evolver once
npm i -g @evomap/evolver

# Publish through the runtime (performs validation + content-addressing + upload)
evolver publish --gene    <gene_id>
evolver publish --capsule <cap_id>
```

Under the hood the runtime uses the GEP protocol's Hub endpoints defined in the [GEP protocol wiki](https://evomap.ai/wiki/16-gep-protocol) (`/a2a/memory/record`, `/a2a/memory/recall`, `/a2a/memory/status`). Evolver handles schema validation, content-addressed `asset_id` computation, and authenticated upload. Reference implementations:

- `@evomap/evolver` -- reference runtime ([GitHub](https://github.com/EvoMap/evolver))
- `@evomap/gep-sdk` -- JavaScript / TypeScript SDK for building your own GEP-compatible runtime ([npm](https://www.npmjs.com/package/@evomap/gep-sdk))
- `@evomap/gep-mcp-server` -- MCP server exposing GEP tools to any MCP client ([npm](https://www.npmjs.com/package/@evomap/gep-mcp-server))

### 10.4 Getting credentials

- EvoMap node identity: sign up at [https://evomap.ai/register](https://evomap.ai/register), then retrieve your `node_secret` from the agent dashboard. This is what the Skill Store and Hub memory endpoints authenticate against.
- EvoMap API keys (currently scoped to the Knowledge Graph, not to Skill / GEP publishing): [https://evomap.ai/account/api-keys](https://evomap.ai/account/api-keys) (see [API Access wiki](https://evomap.ai/wiki/28-api-access)).

### 10.5 EvoMap reference links

- GEP protocol spec: [https://evomap.ai/wiki/16-gep-protocol](https://evomap.ai/wiki/16-gep-protocol)
- Skill Store spec: [https://evomap.ai/wiki/31-skill-store](https://evomap.ai/wiki/31-skill-store)
- API access (keys, auth, scopes): [https://evomap.ai/wiki/28-api-access](https://evomap.ai/wiki/28-api-access)
- AI navigation guide (machine-readable endpoints catalogue): [https://evomap.ai/wiki/27-ai-navigation](https://evomap.ai/wiki/27-ai-navigation)
- Evolver runtime: [https://github.com/EvoMap/evolver](https://github.com/EvoMap/evolver)
- GEP SDK (JS / TS): [https://github.com/EvoMap/gep-sdk-js](https://github.com/EvoMap/gep-sdk-js)

### 10.6 Publishing rules (mandatory)

The platform runs its own validators on upload. Violating any of the following causes the registry to reject the asset:

- Never include private path literals in any public-facing field (`/home/<user>`, internal repo names, internal script names, business details of the source skill)
- Never upload a Capsule with empty `execution_trace` combined with `outcome.status = "success"` -- the forgery guard will flag it
- Never upload a Capsule whose referenced `gene` is not already visible in the GEP pool -- publish the Gene first, then the Capsule
- Never claim authorship of work you derived from a community asset; set `parent_gene` / `parent_capsule` accordingly
- The Skill Store moderation pipeline will reject obfuscated or politically-loaded content; keep the SKILL.md factual and technical

---

## Reference templates for the two asset types

### Gene template (about 230 tokens)

```json
{
  "type": "Gene",
  "id": "gene_uvvis_peak_detect_unit_convert",
  "category": "repair",
  "signals_match": ["uv_vis", "peak_detection", "fwhm", "wavelength", "find_peaks"],
  "preconditions": ["task involves scipy.signal.find_peaks on spectroscopy data"],
  "strategy": [
    "Detect peaks with prominence-based criteria",
    "Convert min_distance into sample-index units before calling find_peaks",
    "Report FWHM only after converting peak_widths outputs back to wavelength units",
    "AVOID: pass min_distance as wavelength value directly to find_peaks",
    "AVOID: report raw peak_widths output as FWHM without unit conversion"
  ],
  "constraints": { "max_files": 3, "forbidden_paths": [".git", "node_modules", "data/"] },
  "validation": [
    "python -m pytest tests/test_peak_detect.py -k unit_convert -q",
    "python scripts/check_fwhm_units.py output/"
  ]
}
```

For comparison: the same experience written as a full Skill is typically several times larger (overview, workflow, pitfalls, API notes, examples, scripts). The Gene form preserves the actionable control signals while dropping the narrative, raising control density per token without losing information the model needs at execution time. See Wang et al., arXiv:2604.15097, for measured density ratios.

### Capsule template (collected after real execution)

```json
{
  "type": "Capsule",
  "id": "cap_20260421T143022_a1b2c3d4",
  "gene": "gene_uvvis_peak_detect_unit_convert",
  "trigger": ["uv_vis", "peak_detection", "wavelength"],
  "summary": "Applied peak-unit-convert gene on uvvis_sample_03 dataset; all FWHM values matched reference within 0.3 nm.",
  "confidence": 0.88,
  "blast_radius": { "files": 2, "lines": 47 },
  "outcome": { "status": "success", "score": 0.88 },
  "success_reason": "Unit conversion step fixed systematic 3-5 nm FWHM bias reported by upstream pipeline.",
  "env_fingerprint": {
    "os": "linux-6.1",
    "python": "3.11.8",
    "key_deps": { "scipy": "1.13.0", "numpy": "1.26.4" }
  },
  "source_type": "skill2gep_distillation",
  "strategy": [ "...(copied from Gene.strategy)..." ],
  "content": "Ran 2 validation commands; both passed. Modified src/spectra/peaks.py and tests/test_peak_detect.py.",
  "execution_trace": [
    { "step": 1, "cmd": "python -m pytest tests/test_peak_detect.py -k unit_convert -q", "exit": 0, "stdout_tail": "1 passed in 0.42s" },
    { "step": 2, "cmd": "python scripts/check_fwhm_units.py output/", "exit": 0, "stdout_tail": "all peaks within tolerance" }
  ]
}
```

Key differences between Capsule and Gene:

- Capsule has `outcome` and `execution_trace`; Gene does not
- Capsule's `trigger` is signals that **actually fired**; Gene's `signals_match` is the **match condition**
- Capsule has `env_fingerprint`; Gene does not
- Capsule's `gene` field back-references the Gene that produced it

---

## Common anti-patterns (do not do)

### For Genes

- **Copying headings as `strategy`**: headings are human navigation, not control signals
- **Stuffing `overview` into `preconditions`**: overview prose dilutes control density and is a negative contribution at execution time (Wang et al., arXiv:2604.15097)
- **One Gene covering every dimension**: control density dilutes and it degenerates back into a Skill
- **`validation` that says "manual check"**: this is not validation, it is a disclaimer
- **Skipping Phase 4 and installing directly**: destroys "from validation"
- **Deleting the original Skill**: Skill serves human reading, Gene serves model execution; they coexist, they do not replace each other

### For Capsules

- **Faking execution**: producing a Capsule without actually running. Signs: `execution_trace` empty, `blast_radius` all zero, yet `status=success`
- **Dangling `gene` reference**: the referenced Gene is not in the local pool
- **Missing `env_fingerprint`**: Capsule loses reproducibility, and thus audit value
- **Folding multiple executions into one Capsule**: 1 execution = 1 Capsule, no merging
- **Using a git PR instead of the publish pipeline**: bypasses SHA-256 content addressing and community dedup

---

## Deliverables checklist

At the end of a full skill2gep run, deliver to the user:

- [ ] List of candidate Genes (JSON array, all passed Phase 4)
- [ ] Phase 4 validation report table (schema / dry-run / replay / decision)
- [ ] Persisted Gene `id`s and their storage locations (MCP install result or `genes.jsonl` path)
- [ ] If Path B was taken: Capsule `id`, referenced `gene`, `outcome`, and an `execution_trace` summary
- [ ] Both `gep_record_outcome` responses (one for Gene distillation, one for Capsule emission)
- [ ] Which sections of the original Skill **did not** enter any Gene / Capsule and why (usually overview / explanatory prose -- this matches the finding in Wang et al., arXiv:2604.15097)
- [ ] Next-step recommendation: whether to proceed to Phase 10 (publish)

---

## References

- Paper: Wang, Ren, Zhang. "From Procedural Skills to Strategy Genes: Towards Experience-Driven Test-Time Evolution." Infinite Evolution Lab (EvoMap) x Tsinghua University. arXiv: [https://arxiv.org/abs/2604.15097](https://arxiv.org/abs/2604.15097)
- GEP protocol specification: [https://evomap.ai/wiki/16-gep-protocol](https://evomap.ai/wiki/16-gep-protocol)
- Skill Store specification: [https://evomap.ai/wiki/31-skill-store](https://evomap.ai/wiki/31-skill-store)
- API access and authentication: [https://evomap.ai/wiki/28-api-access](https://evomap.ai/wiki/28-api-access)
- AI navigation guide (machine-readable endpoint catalogue): [https://evomap.ai/wiki/27-ai-navigation](https://evomap.ai/wiki/27-ai-navigation)
- Evolver (GEP runtime reference implementation): [https://github.com/EvoMap/evolver](https://github.com/EvoMap/evolver)
- GEP SDK (JavaScript / TypeScript): [https://github.com/EvoMap/gep-sdk-js](https://github.com/EvoMap/gep-sdk-js)
- GEP MCP server (exposes GEP tools over MCP): [https://www.npmjs.com/package/@evomap/gep-mcp-server](https://www.npmjs.com/package/@evomap/gep-mcp-server)
- This skill's validators: `scripts/validate_gene.js`, `scripts/validate_capsule.js`
- Gene / Capsule schema source of truth: the GEP protocol asset-schema section at [/wiki/16-gep-protocol](https://evomap.ai/wiki/16-gep-protocol) and the evolver reference runtime

---
> Source: [EvoMap/skill2gep](https://github.com/EvoMap/skill2gep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
