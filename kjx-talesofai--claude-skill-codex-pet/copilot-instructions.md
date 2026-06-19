## claude-skill-codex-pet

> Create, repair, validate, preview, and package Codex-compatible animated pet spritesheets from character art, screenshots, generated images, or visual references. Use when a user wants to hatch a Codex pet, create a custom animated pet, or build a built-in pet asset with an 8x9 atlas, transparent unused cells, row-by-row animation prompts, QA contact sheets, preview videos, and pet.json packaging. Requires an image generator with text-prompt + reference-image grounding capabilities. Uses bundled deterministic scripts for spritesheet assembly.


# Hatch Pet

## Overview

Create a Codex-compatible animated pet from a concept, one or more reference images, or both. This skill owns pet-specific prompt planning, animation rows, frame extraction, atlas geometry, QA, previews, and packaging. It delegates visual generation to whatever image generation tool is available in the current environment.

User-facing inputs are optional. If the user omits a pet name, infer one from the concept or reference filenames; if that is not possible, choose a short appropriate name. If the user omits a description, infer one from the concept or references. If the user omits reference images, generate the base pet from text first, then use that base as the canonical reference for every animation row.

## Required Image Generation Capabilities

The image generator used with this skill must support:

1. **Text-to-image generation** from detailed prompts.
2. **Reference image grounding** (attaching input images to guide output). This is required for row-strip generation — every row after the base must use grounding images to preserve identity.
3. **PNG output** (or convertible to PNG). The deterministic pipeline ingests PNGs.
4. **Reasonable prompt adherence** for precise layouts: the generator should be able to produce a horizontal strip with an exact frame count, flat green background, and evenly spaced poses when instructed.

If the available image generator does not support reference image grounding, it can only be used for the base job (which may be prompt-only). All row-strip jobs require grounding.

**Tested providers:** This skill has been tested with GPT Image 2 via proxy (strong reference grounding, URL output) and Google Gemini image generation (text-only for base, no reference support for rows). Any image generator meeting the four capabilities above should work.

## Generation Delegation

Use the environment's available image generation tool for all visual generation.

When invoking an image generator from this skill, pass the generated pet prompt as the authoritative visual spec. Do not wrap it in the generic shared prompt schema and do not add extra polish, hero-art, photo, product, or illustration-style augmentation. Pet prompts should stay terse, sprite-specific, and digital-pet oriented; only add role labels for input images and any essential user constraint.

Use this skill's scripts for deterministic work only: preparing prompts and manifests, ingesting selected image generator outputs, extracting frames, validating rows, composing the final atlas, creating QA media, and packaging.

Hard boundary: do not create, draw, tile, warp, mirror, or synthesize pet visuals with local Python/Pillow scripts, SVG, canvas, HTML/CSS, or other code-native art as a substitute for the image generator. For a normal pet run, expect up to 10 visual generation jobs: 1 base pet plus 9 row-strip jobs. The only exception is `running-left`, which may be derived by mirroring `running-right` only after `running-right` has been generated, visually inspected, and explicitly approved as safe to mirror. If mirroring is not appropriate, generate `running-left` as a normal grounded row. If those calls are too expensive, blocked, or unavailable, stop and explain the blocker instead of fabricating row strips locally.

Do not mark visual jobs complete by editing `imagegen-jobs.json`, copying files into `decoded/`, or writing helper scripts that populate row outputs. Use `record_imagegen_result.py` for selected image generator outputs. The deterministic scripts may only process already-generated visual outputs.

Only the base job may be prompt-only. Every row-strip job must use the input images listed in `imagegen-jobs.json`, including the canonical base reference created after the base job is recorded. Treat any row generation without attached grounding images as invalid.

## Image Generation Integration Notes

### Discovering the local image generator

Before generating, check what image generation tools are available in the current environment. Common patterns:

- Look for a skill or command in the agent's skill directory (e.g., `~/.claude/skills/`, `.claude/skills/`)
- Check for CLI tools like `nb-generate`, `gpt-image`, or custom wrappers
- Check environment variables like `OPENAI_API_KEY`, `GPT_IMAGE_API_KEY`

### Handling URL outputs

If the image generator returns a URL instead of writing a local file:

1. Download the image from the URL.
2. Save it as a local PNG.
3. Pass the local PNG path to `record_imagegen_result.py`.

### Enforcing frame consistency in prompts

Frame size and identity drift between frames is the most common quality issue. Every row prompt should include **explicit consistency constraints** tailored to the animation state:

**For all rows:**
- "ALL frames must show the character at EXACTLY THE SAME BODY SIZE and SAME POSITION in every frame."
- "NO body breathing motion, NO size changes, NO shifting between frames unless the action specifically requires it."
- "Each frame must be evenly spaced and centered in its invisible slot."

**State-specific constraints:**
- `idle`: "The ONLY change across frames is eye blinking. Body, head, hair, outfit, limbs must remain IDENTICAL."
- `waiting`: "ALL frames show the character STANDING upright. ONLY subtle head turns or small foot taps. NO sitting, NO waving, NO dramatic pose changes."
- `waving`: "Same body size in all frames. ONLY the waving arm/paw changes position."
- `jumping`: "Same body size. Character moves vertically through jump phases — body position changes but silhouette size stays consistent."
- `running-right` / `running-left` / `running`: "Same body size. ONLY limb positions change for gait cycle."
- `failed`: "Same body size. Expression and posture sag subtly but body dimensions stay constant."
- `review`: "Same body size. ONLY head tilt, eye focus, or lean changes."

If a generated row shows frame size variation greater than ~2 pixels, inconsistent poses, or unexpected state bleed (e.g., sitting in a waiting row), regenerate with stronger constraints.

### Handling reference images

If the image generator supports local reference images, resize them to ~500px max dimension before sending to keep payloads small and avoid timeouts. The reference images for each row are listed in `imagegen-jobs.json` and include:

- The original user reference (if any)
- `references/canonical-base.png` (the approved base pet)
- `references/layout-guides/<state>.png` (frame layout guide)

Treat layout guides as invisible construction references — the output must not include visible boxes, guide lines, labels, or guide colors.

## Codex Digital Pet Style

Default pet art should match the Codex app's built-in digital pets: small pixel-art-adjacent mascots with compact chibi proportions, chunky readable silhouettes, thick dark 1-2 px outlines, visible stepped/pixel edges, limited palettes, flat cel shading, simple expressive faces, and tiny limbs. Even if the reference art is more detailed, complex or realistic, the generated pet should be simplified into this style.

Do NOT generate polished illustration, painterly rendering, anime key art, 3D rendering, glossy app-icon treatment, realistic fur or material texture, soft gradients, high-detail antialiasing, and complex tiny accessories. References that are more detailed than this should be simplified into the house style before row generation.

## Transparency And Effects

Pet rows are processed into transparent 192x208 cells, so every generated pixel must either belong to the pet sprite or be cleanly removable chroma-key background. Prefer pose, expression, and silhouette changes over decorative effects.

Allowed effects must satisfy all of these conditions:

- The effect is state-relevant and helps explain the animation.
- The effect is physically attached to, touching, or overlapping the pet silhouette, not floating nearby.
- The effect is inside the same frame slot as the pet and does not create a separate sprite component.
- The effect is opaque, hard-edged, pixel-style, and uses non-chroma-key colors.
- The effect is small enough to remain readable at 192x208 without clutter.

Examples of allowed effects: a tear touching the face, a small smoke puff touching the box or head, or tiny stars overlapping the pet during a failed/dizzy reaction.

Avoid these by default because they usually break transparent-background cleanup or component extraction:

- wave marks, motion arcs, speed lines, action streaks, afterimages, blur, or smears
- detached stars, loose sparkles, floating punctuation, floating icons, falling tear drops, separated smoke clouds, or loose dust
- cast shadows, contact shadows, drop shadows, oval floor shadows, floor patches, landing marks, impact bursts, glow, halo, aura, or soft transparent effects
- text, labels, frame numbers, visible grids, guide marks, speech bubbles, thought bubbles, UI panels, code snippets, checkerboard transparency, white backgrounds, black backgrounds, or scenery
- chroma-key-adjacent colors in the pet, prop, effects, highlights, or shadows
- stray pixels, disconnected outline bits, speckle/noise, cropped body parts, overlapping poses, or any pose that crosses into a neighboring frame slot

State-specific guidance:

- `waving`: show the wave through paw pose only. Do not draw wave marks, motion arcs, lines, sparkles, or symbols around the paw.
- `jumping`: show vertical motion through body position only. Do not draw shadows, dust, landing marks, impact bursts, bounce pads, or floor cues.
- `failed`: tears, attached smoke puffs, or attached stars are allowed if they obey the allowed-effects rules; do not use red X marks, floating symbols, detached smoke, detached stars, or separate tear droplets.
- `review`: show focus through lean, blink, eyes, head tilt, or paw position. Do not add magnifying glasses, papers, code, UI, punctuation, or symbols unless that prop already exists in the base pet identity.
- `running-right`, `running-left`, and `running`: show locomotion through body, limb, and prop movement only. Do not draw speed lines, dust clouds, floor shadows, or motion trails.

## Pet Naming

Ask the user for a pet name when they have not provided one and only if the conversation naturally allows it. If asking would slow down a direct execution request, choose a short appropriate name from the pet concept, reference image, or personality, then use that name consistently as the display name and as the source for the package folder slug.

Good built-in style examples:

- Codex - The original Codex companion.
- Dewey - A tidy duck for calm workspace days.
- Fireball - Hot path energy for fast iteration.
- Rocky - A steady rock when the diff gets large.
- Seedy - Small green shoots for new ideas.
- Stacky - A balanced stack for deep work.
- BSOD - A tiny blue-screen gremlin.
- Null Signal - Quiet signal from the void.

## Visible Progress Plan

For every pet run, keep a visible checklist so the user can see where the work is up to. Create the checklist before starting, keep one step active at a time, and update it as each step finishes.

Before creating the checklist, establish the pet name when possible. Use the user-provided name when available; otherwise infer a short appropriate name from the concept or references. If the name is too long, not settled, or not appropriate for a friendly checklist, use `your pet` instead.

Use this checklist for a normal pet run, replacing `<Pet>` with the pet's name or `your pet`:

1. Getting `<Pet>` ready.
2. Imagining `<Pet>`'s main look.
3. Picturing `<Pet>`'s poses.
4. Hatching `<Pet>`.

What each step means:

- `Getting <Pet> ready.` Choose or confirm the pet name, description, source images, and working folder.
- `Imagining <Pet>'s main look.` Generate the pet's main reference image. This is required for new pets, even when the user does not provide an image, because it becomes the visual source of truth.
- `Picturing <Pet>'s poses.` Create the pose rows, starting with `idle` and `running-right` to confirm the pet still looks consistent. Only mirror `running-left` if `running-right` clearly works when flipped.
- `Hatching <Pet>.` Turn the approved poses into the final pet files, review the contact sheet, previews, and validation results, fix any broken parts, save `pet.json` and `spritesheet.webp` into the pet folder, then tell the user where the pet and QA files were saved.

Only mark a step complete when the real file, image, or decision exists. If this is just a repair run, start from the first relevant step instead of restarting the whole checklist.

## Default Workflow

1. Prepare a pet run folder and imagegen job manifest:

```bash
SKILL_DIR="<path-to-this-skill>"
python "$SKILL_DIR/scripts/prepare_pet_run.py" \
  --pet-name "<Name>" \
  --description "<one sentence>" \
  --reference /absolute/path/to/reference.png \
  --output-dir /absolute/path/to/run \
  --pet-notes "<stable pet description>" \
  --style-notes "<style notes>" \
  --force
```

All arguments above are optional except any flags needed to express user constraints. For text-only requests, pass the concept through `--pet-notes` and omit `--reference`; `prepare_pet_run.py` will infer a name, description, chroma key, and output directory as needed.

2. Inspect the next ready image generation jobs:

```bash
python "$SKILL_DIR/scripts/pet_job_status.py" --run-dir /absolute/path/to/run
```

3. For each ready job, invoke the available image generator with:

- the prompt file listed in `imagegen-jobs.json`
- every input image listed for the job, with its role label

The base job must complete first. If user references exist, the base job uses them. If no references exist, the base job may be prompt-only. After recording the base, `record_imagegen_result.py` writes `decoded/base.png` and `references/canonical-base.png`; all row jobs use the original references if present plus those canonical base images.

`prepare_pet_run.py` also creates 9 row-specific layout guide images under `references/layout-guides/`, one per animation state. Row jobs attach the matching guide as a layout-only input so the model can follow the correct frame count, spacing, centering, and safe padding. Treat these guides as invisible construction references: the generated row strip must not include visible boxes, borders, center marks, labels, guide colors, or the guide background.

When generating row strips, keep the identity lock in the row prompt authoritative: do not redesign the pet, and preserve the same head shape, face, markings, palette, prop, outline weight, body proportions, and silhouette. A row that looks like a related but different pet is failed even if the deterministic geometry QA passes.

Generate and record `running-right` before deciding how to complete `running-left`. Inspect `running-right` against the base and references. If the pet is visually symmetric enough that a horizontal mirror preserves identity, prop placement, handedness, markings, lighting, text-free details, and direction semantics, derive `running-left` with:

```bash
python "$SKILL_DIR/scripts/derive_running_left_from_running_right.py" \
  --run-dir /absolute/path/to/run \
  --confirm-appropriate-mirror \
  --decision-note "<why mirroring preserves this pet's identity>"
```

If there is any asymmetric side-specific marking, readable text, non-mirrored logo, handed prop, one-sided accessory, lighting cue, or direction-specific pose that would become wrong when flipped, do not mirror. Generate `running-left` with the image generator using its row prompt and all listed grounding images, including `decoded/running-right.png` as a gait reference.

4. After selecting a generated output for a job, ingest it:

```bash
python "$SKILL_DIR/scripts/record_imagegen_result.py" \
  --run-dir /absolute/path/to/run \
  --job-id <job-id> \
  --source /absolute/path/to/generated-output.png \
  --allow-synthetic-test-source
```

This copies the image to the exact decoded path expected by the deterministic pipeline and records source metadata in `imagegen-jobs.json`.

> **Why `--allow-synthetic-test-source`?** The original Codex version of this skill hardcodes a requirement that every source must live under a Codex-specific output directory (`~/.codex/generated_images/.../ig_*.png`). When using non-Codex image generators that write to their own output directories or return URLs, this provenance check blocks ingestion. The `--allow-synthetic-test-source` flag (already built into the scripts) relaxes this check so any image generator's output can flow through the pipeline.

5. When all jobs are complete, finalize with local output:

```bash
python "$SKILL_DIR/scripts/finalize_pet_run.py" \
  --run-dir /absolute/path/to/run \
  --allow-synthetic-test-sources \
  --package-dir /absolute/path/to/output/pets/<pet-name>
```

Expected output in the run directory:

```text
run/
  pet_request.json
  imagegen-jobs.json
  prompts/
  decoded/
  frames/frames-manifest.json
  final/spritesheet.png
  final/spritesheet.webp
  final/validation.json
  qa/contact-sheet.png
  qa/review.json
  qa/run-summary.json
  qa/videos/*.mp4
```

Packaged pet output (via `--package-dir`):

```text
<package-dir>/
  pet.json
  spritesheet.webp
```

> **Backward compatibility:** The scripts still accept `CODEX_HOME` if you want the original Codex behavior. Set `CODEX_HOME` and omit `--package-dir` to output to `${CODEX_HOME}/pets/<pet-name>/`.

Review `qa/contact-sheet.png`, `qa/review.json`, `final/validation.json`, and `qa/videos/` before accepting the pet.

Deterministic validation is necessary but not sufficient. Before calling the pet done, visually inspect the contact sheet for identity consistency. Block acceptance if any row changes species/body type, face, markings, palette, prop design, prop side unexpectedly, or overall silhouette.

## Subagent Row Generation

After the base job has been recorded and `references/canonical-base.png` exists, row-strip visual generation should use subagents unless the user explicitly says not to use subagents for this session. Before row generation, state that subagents are being used and which row jobs are being delegated. If subagents cannot be spawned because the current environment or tool policy blocks them, stop before row-strip generation, explain the blocker, and ask for explicit user direction before continuing sequentially.

The parent agent must own the manifest and package writes.

Default flow:

1. Parent runs `prepare_pet_run.py`.
2. Parent generates and records `base`.
3. Parent runs `pet_job_status.py`.
4. Parent spawns subagents for `idle` and `running-right` first as identity and gait checks.
5. Parent records the selected `idle` and `running-right` results returned by subagents.
6. Parent decides whether `running-left` is safe to derive by mirror; if not, parent treats it as a normal grounded row job delegated to a subagent.
7. Parent spawns subagents for every remaining non-derived row image-generation job.
8. Each subagent receives the row prompt and every listed input image path, invokes the environment's image generator, and returns only the selected output file path.
9. Parent alone runs `record_imagegen_result.py`, `derive_running_left_from_running_right.py`, repair queueing, finalization, QA, and packaging.

Subagent write boundary: do not let subagents edit `imagegen-jobs.json`, copy files into `decoded/`, run `record_imagegen_result.py`, run `derive_running_left_from_running_right.py`, run `finalize_pet_run.py`, or package the pet. This avoids manifest races and keeps provenance checks centralized.

Subagent handoff contract:

- Give each subagent exactly one row job unless you are intentionally batching adjacent simple rows.
- Include the row id, the absolute prompt file path, the full prompt text or an instruction to read that exact prompt file, and every input image path with its role label from `imagegen-jobs.json`.
- Explicitly remind the subagent that the prompt's transparency and effects rules are mandatory: no detached effects, no wave marks for `waving`, no speed lines or dust for running rows, and only attached opaque sprite-like tears/smoke/stars when allowed by the state prompt.
- Tell the subagent to inspect the generated candidate for frame count, identity consistency, clean flat chroma-key background, safe spacing, and forbidden detached effects before returning it.
- Tell the subagent to return only the selected output file path plus a one-sentence QA note. The parent decides whether to record or repair it.

Use this template for each subagent:

```text
Generate the `<row-id>` row for this hatch-pet run.

Run dir: <absolute run dir>
Prompt file: <absolute prompt file>
Input images:
- <absolute path> — <role>
- <absolute path> — <role>

Read and follow the row prompt exactly, including the Transparency and artifact rules. Use the environment's image generation tool only; do not use local scripts to draw, tile, edit, or synthesize sprites.

Steps:
1. Call the image generator with the row prompt and all attached reference images.
2. If the output is a URL, download it to a local PNG.
3. Visually verify: exact frame count, same identity as base, green background, no detached effects.

Do not edit manifests, copy into decoded, record results, mirror rows, finalize, repair, or package. Return only:
selected_source=/absolute/path/to/the/generated.png
qa_note=<one sentence>
```

No silent sequential fallback: if subagents cannot be used for row-strip visual generation, stop and ask for explicit user direction before continuing without them. Only an explicit user instruction such as "do not use subagents" or "run this sequentially" authorizes a normal sequential row-generation path. The final answer must report which row jobs were delegated to subagents and which, if any, were mirrored or repaired by the parent.

## Repair Workflow

If finalization stops because row QA failed, queue targeted repair jobs:

```bash
python "$SKILL_DIR/scripts/queue_pet_repairs.py" \
  --run-dir /absolute/path/to/run
```

Then repeat the image generation and `record_imagegen_result.py` ingest loop for each reopened row job. Regenerate the smallest failing scope: the failed row, not the whole sheet.

For identity repairs, use the canonical base image, original references, contact sheet, and exact row failure note as grounding context. Repair only the failed row while preserving the canonical pet identity.

## Secondary Image Generation Fallback

`scripts/generate_pet_images.py` is a secondary fallback that calls the OpenAI Images API directly (`OPENAI_API_KEY` required). It is available in this repository unchanged from the original Codex skill.

Use the secondary fallback only when the environment's primary image generator is unavailable:

```bash
python "$SKILL_DIR/scripts/generate_pet_images.py" \
  --run-dir /absolute/path/to/run \
  --model gpt-image-2 \
  --states all
```

The secondary fallback requires `OPENAI_API_KEY`.

## Rules

- Use the environment's available image generator as the primary generation layer.
- Keep reference images attached/visible for the image generator whenever the chosen path supports references.
- Attach the row's `references/layout-guides/<state>.png` image to every row-strip job as a layout-only guide, and do not accept outputs that copy guide pixels.
- Use subagents for row-strip visual generation after the parent records the base image. The parent may generate the base, but row-strip jobs belong to subagents unless the user explicitly says not to use subagents for this session.
- Generate every normal visual job with the image generator: base plus all row strips that are not explicitly approved `running-left` mirror derivations.
- Treat only the base job as eligible for prompt-only generation; every row job must attach its listed grounding images.
- Delegate `running-right` first, then mirror `running-left` only when visual inspection confirms a mirror preserves identity and semantics; otherwise delegate `running-left` as a normal grounded row.
- Never substitute locally drawn, tiled, transformed, or code-generated row strips for missing image generator outputs.
- Never manually mutate `imagegen-jobs.json` to claim a visual job completed.
- Do not rely on generated images for exact atlas geometry; use this skill's deterministic scripts.
- Use the chroma key stored in `pet_request.json`; do not force a fixed green screen.
- Keep the pet's silhouette, face, materials, palette, and props consistent across all rows.
- Enforce the transparency and effects rules above in every base, row, and repair prompt.
- Treat visual identity drift as a blocker even when `qa/review.json` and `final/validation.json` have no errors.
- Treat a contact sheet that shows cropped references, repeated tiles, white cell backgrounds, or non-sprite fragments as failed.
- Treat forbidden detached effects, chroma-key-adjacent artifacts, shadows, glows, smears, dust, landing marks, wave marks, speed lines, or motion trails as failed rows.
- Treat `qa/review.json` errors as blockers. Warnings require visual review.

## Acceptance Criteria

- Final atlas is PNG or WebP, `1536x1872`, transparent-capable, and based on `192x208` cells.
- Used cells are non-empty and unused cells are fully transparent.
- Atlas follows the row/frame counts in `references/animation-rows.md`.
- Contact sheet and preview videos have been produced unless explicitly skipped.
- `qa/review.json` has no errors.
- Row-by-row review confirms the animation cycles are complete enough for the Codex app.
- `pet.json` and `spritesheet.webp` are staged together in the package directory (default: local `./output/pets/<pet-name>/`).

---
> Source: [kjx-talesofai/claude-skill-codex-pet](https://github.com/kjx-talesofai/claude-skill-codex-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
