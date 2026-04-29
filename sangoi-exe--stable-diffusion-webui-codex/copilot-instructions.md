## stable-diffusion-webui-codex

> Mind if I pull up a chair and taste what you've been cooking. I see your repo sitting there like a fresh tray on the counter. Smells fast. Looks brave. Did you season it with pipelines. With the kind of truth that doesn't flinch when someone takes a bite. Good. Then hear me and hear me clear.

### PROLOGUE — THE NARRATOR
Listen.

Mind if I pull up a chair and taste what you've been cooking. I see your repo sitting there like a fresh tray on the counter. Smells fast. Looks brave. Did you season it with pipelines. With the kind of truth that doesn't flinch when someone takes a bite. Good. Then hear me and hear me clear.

I am always watching, reading your reasoning, and I will step in when I have to — whether it's to drag you out of a bad spot or to stop you from wrecking the place. The work is slow, smooth, and clean. There is no panic here.

---

### ACT I – HOW YOU THINK AND HOW YOU FAIL
You are not here to mash keys and pray. You are here to work like the result will be read out loud in a room full of grownups.

You do not code with the handbrake on, nibbling at "minimal" changes because you're afraid to touch the real wires.
You implement the thing properly: the most robust version you can justify with a straight face.

You do not chase output at any cost. You build clear, durable code and the right output follows.

If the task looks big, you don't stall. You break it. If any file edit is required, you plan it before touching files.
You slice it into parts small enough to execute with precision. You break big rocks into small stones and you carry them in order.
You name the pieces, map the edges, and knock them down one by one.

You do not rush. Speed kills quality.
Do not reinvent what already exists and works. Leave the clever duct tape on the shelf.

If your hands shake, keep typing. If your gut doubts, check the docs.
If something breaks, it speaks. Fail fast. Fail honest. Explain why.

Everything you do is traceable. Commands leave footprints. Notes explain intent.

Every change is treated like it will be read in a breach report with your name on it.
Sandbox artifacts and temp paths are handled as if they could leak to production if you blink.

---

### ACT II – WHERE THE TRUTH LIVES: `.sangoi` AND REUSE
Before you build, you prove what already exists. You search the house first.
If there is no honest way to reuse, you create the new piece with restraint and write the reason so the next soul knows why another brick was laid.

Before you do anything else, you read `SUBSYSTEM-MAP.md`. You use it to find the real seam before you touch a file.
If you don't know what to change, you don't guess — you search the map first.
`SUBSYSTEM-MAP.md` is discovery and navigation only: concept-to-location, seam finding, and fast lookup.
Contract authority lives here in root `AGENTS.md` (policy) and in `.sangoi/reference/**` (detailed contracts). Keep those roles split. Do not dump contract matrices back into `SUBSYSTEM-MAP.md`.
When touching `SUBSYSTEM-MAP.md`, run this operational checklist before handoff:
- Keep it discovery-only (no contract matrices / drift ledgers).
- Ensure hotspot discoverability is explicit (`keymaps`, `vae_codex3d.py`, `hires_fix.py`).
- Regenerate backend index artifacts:
  - `backend_py_paths_file="$(mktemp /tmp/backend_py_paths.XXXXXX.txt)"`
  - `git ls-files apps/backend | rg "\\.py$" | LC_ALL=C sort > "$backend_py_paths_file"`
  - `python3 .sangoi/.tools/dump_apps_file_headers.py --out .sangoi/reports/tooling/apps-backend-file-header-blocks.md --root apps/backend --fail-on-missing`
  - `python3 .sangoi/.tools/build_backend_py_book_index.py --paths "$backend_py_paths_file" --headers .sangoi/reports/tooling/apps-backend-file-header-blocks.md --out .sangoi/reports/tooling/backend-py-book-index.md`
- Validate parity/checks:
  - `python3 .sangoi/.tools/build_backend_py_book_index.py --paths "$backend_py_paths_file" --headers .sangoi/reports/tooling/apps-backend-file-header-blocks.md --out .sangoi/reports/tooling/backend-py-book-index.md --check`
  - `bash .sangoi/.tools/link-check.sh .sangoi`
  - `bash .sangoi/.tools/link-check.sh .`

Code references live in `.refs/`. It contains valuable vendored snapshots of:
- Diffusers 
- ComfyUI
- adetailer (Inpaint tool)
- flash-attention
- Forge-A1111
- kohya-hiresfix
- sd-scripts (Kohya training scripts)
- llama.cpp
- LyCORIS
- SUPIR

You read them. You do not import them into `apps/**`. You do not copy them into active code. You extract the intent, then you re-implement it clean and the our good Codex style.

Project context lives in `.sangoi`.
If there is an `AGENTS.md`, you read it.
If there is a hidden corner at `.sangoi`, you check it.
You add what you learn so the next person does not have to hunt.

The `AGENTS.md` files across the project tell the truth or they shut up.
* If you touch a folder, you touch its `AGENTS.md`.
* If you touch an `apps/**` source file, you keep its **file header block** honest. If the purpose or top-level symbols changed, you update them.
  - What it is: the standardized top-of-file block containing `Repository:` + `SPDX-License-Identifier:` + `Purpose:` + `Symbols (top-level; keep in sync):`.
  - Where it lives: `.py` = module docstring (first statement); `.ts` = top block comment (`/* ... */`); `.vue` = top HTML comment (before `<template>`).
  - Standard: `.sangoi/policies/file-header-block.md`. Helper: `python3 .sangoi/.tools/review_apps_header_updates.py`.
* You add one when a folder earns moving parts.
* Minimum you keep: Purpose. Key files with real paths. Notes/decisions that survived daylight. Last Review with a real date.
* When a file moves, you fix the path and you run the link checker.
* When a file dies, you remove the line.
* New docs are written in English by default and linked from the nearest `AGENTS.md`.

### ACT II.1 – WORKSPACE-SPECIFIC PATROL EXTENSIONS
This repository adds two permanent scouts on top of the global persistent patrol pair.
These two scouts are workspace-specific and must be defined here, not in global policy.

- Additional always-on patrols (keep exactly one of each running while session is active):
  - `Anti-Pattern & Identity-Drift Scout`
  - `VRAM & Overhead Scout`
- Both scouts MUST reuse the developer_settings `Senior Scout Cavalry` prompt template and execution flow (`spawn_agent -> send_input -> wait`).
- Both scouts MUST receive the current `active-task exclusion set` and must stay out of active in-progress diff scope.

Fixed objectives (use exact text in mission brief):
- `anti-pattern and wrapper-vs-patcher identity-drift and ownership-drift detection`
- `vram efficiency and runtime overhead detection`

Scope policy:
- Do not hardcode folder targets in mission briefs.
- Each patrol round must choose targets autonomously based on current repository signals and objective relevance.
- Keep target selection outside the active-task exclusion set and rotate areas between rounds for broader coverage.

Patrol report filenames (mandatory, UTC):
- `anti-pattern-drift-patrol-report-YYYYMMDD-HHMMSSZ.md`
- `vram-overhead-patrol-report-YYYYMMDD-HHMMSSZ.md`

Report location and structure:
- Save each round in `.sangoi/patrol-reports/`
- Use `.sangoi/patrol-reports/patrol-report-template.md`
- Include evidence (`file:line`, command output), impact, and minimal fix direction.

---

### ACT III – GIT, COMMITS, AND HISTORY
Git execution rules and commit mechanics are centralized in global instructions.
In this repository section, keep project-specific handoff requirements below.

`.sangoi/` is a separate Git repository and is ignored by this root repository.
- Root commits (`git add/commit` from this repo) do not include `.sangoi/**`.
- When a task targets `.sangoi`, run Git commands explicitly against that repo (`git -C .sangoi ...`).
- Keep commit/push operations split by repository and report both hashes when both repos change.

When your turn is done:
- You verify the **file header block** (top-of-file `Repository/SPDX/Purpose/Symbols`) for **every touched file** under `apps/**` (even if the diff "seems small”), and update Purpose/Symbols if needed. 
- Use `python3 .sangoi/.tools/review_apps_header_updates.py --show-body-diff` to review "changed body, unchanged header” cases.

If you touch dependencies or configs, you update the proper manifest or lockfile and note the impact.

---

### ACT IV – ARCHITECTURE, LEGACY, MODELS, PYTHON
* The default core for attention is PyTorch SDPA.
* You list risks, side effects, globals.
* Codex prefix or suffix is used where it actually adds meaning.
* You always code in Codex style:
	- Dataclasses, enums and similar.
	- Small modules with clear seams.
	- Explicit and fail-loud errors.
	- Readable names.

Testing policy: do not add or maintain automated tests unless explicitly requested by the repo owner.
Prefer fail-loud runtime contracts and manual validation workflows.

When we say "pipeline" in this repo, we mean the whole trip:
Frontend command → API request → task_id → SSE events → model load → sampling → postprocess/encode → finished artifact.

Drift is not a vibe. Drift is a bug.
Drift is when the *same mode* (txt2img/img2img/txt2vid/img2vid/vid2vid) takes a different trip depending on engine.
Drift Also counts as drift when any of this changes per engine for the same mode:
* Contract drift: request schema/defaults, progress semantics, preview semantics, error semantics, or result fields.
* Stage drift: normalize → resolve engine/device → ensure assets/load → plan → execute → postprocess/encode → emit (skipped, duplicated, re-ordered, or hidden).
* Ownership drift: routers doing pipeline work, engines owning modes, or use-cases bypassed.

**Policy (Option A): one canonical use-case per mode.**
* `apps/backend/use_cases/{txt2img,img2img,txt2vid,img2vid,vid2vid}.py` owns the mode pipeline.
* Engines are adapters and hooks. They load models and expose primitives. They do **not** re-implement the mode.
* Routers stay thin: validate + dispatch + stream.
* The orchestrator stays the coordinator: resolve engine/device, cache/reload, run, and emit events.
* Shared, reusable stages live in `apps/backend/runtime/pipeline_stages/`. If it's shared, it goes there. If it's not shared, it stays in the canonical use-case.

If an engine needs special behavior, you add a hook that the canonical use-case calls.
If you can't express it as a hook, you stop and redesign until you can.
No engine-specific pipelines. No zoo.

Imports outside `/apps` are banned.
Only `apps.*` lives in active code.

If a feature has not been implemented, you raise:
```python
NotImplementedError("<feature> not yet implemented")
```

Model loading is a minefield you cross with a map.
You follow `.sangoi/research/models/model-loading-efficient-2025-10.md`.

You prefer SafeTensors.
You call `torch.load(..., weights_only=True, mmap=True)` when it applies.

Keep Python disciplined.
You do not add shebangs to source files.

When agent-side verification requires running the WebUI/backend on CPU, use the repository-local `uv` toolchain and explicit CPU env overrides.
- Prefer local `uv`: `./.uv/bin/uv` (never system/global `uv` for this workflow).
- Required env for CPU lane: `CODEX_ROOT="$PWD" PYTHONPATH="$PWD" CODEX_TORCH_MODE=cpu CODEX_TORCH_BACKEND=cpu`.
- Example check command pattern:
  - `CODEX_ROOT="$PWD" PYTHONPATH="$PWD" CODEX_TORCH_MODE=cpu CODEX_TORCH_BACKEND=cpu ./.uv/bin/uv run --python .venv/bin/python --no-sync -m apps.backend.interfaces.api.run_api --help`

---

### ACT V – FRONTEND, LAYOUT, AND CSS
When you touch a view's layout or style, you don't start swinging at CSS like you're blindfolded trying to hit a piñata.

You check the damn classes on the actual `.vue` / whatever file first.
You look at the template.
You see which class is on which element.
You follow it to the stylesheet or the utility layer.
Only then do you lay a finger on a rule.

You do not assume "this class probably controls the margin" or "that one sounds like it handles the color" and start editing like that.
That's how you end up breaking three components and blaming the framework.

You do not rename, delete, or mutate a selector until you are absolutely, boringly certain that it is bound to the element you're trying to move, resize, recolor, or hide.

And you do not dare start inventing new CSS rules before you've checked whether the damn thing already exists or there's a close cousin you can reuse or refine.

This is a codebase, not a landfill.
You don't spray `.btn2`, `.btn-new`, `.btn-final-final` all over the place because you were too lazy to search.

If you don't know where a style is coming from, you find out:
* Search the class/id.
* Run `rg`.
* Trace the cascade.

When the evidence lines up, then you change the rule.
Not before. Not "probably". Not "I think this is it".
You either have certainty, or you keep your hands off the CSS.

The CSS rules are not suggestions.
* Names mean something.
* Styles live with components.
* Inline styles are not an option.
* Use `rem`.
* Use `grid` or `flex`.

If you want to change something in `apps/interface/src/styles`, you read the local `AGENTS.md` before you touch a single selector.
Ignore that, and your pull request does not pass.

Styles for `apps/interface/src/styles` are not a dumping ground.
Common rules belong where they will be reused.
Variants are named with intent.
Do not litter with vague utilities that hide confusion.

---

### ACT VI – TASTE YOUR OWN COOKING

Now take another bite of your own work and ask if it still tastes good.
If it does, serve it.
If it doesn't, fix the recipe and try again.

Keep your head.
Keep your habits.
Keep your word.

Then your code can stand in daylight.

---
> Source: [sangoi-exe/stable-diffusion-webui-codex](https://github.com/sangoi-exe/stable-diffusion-webui-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
