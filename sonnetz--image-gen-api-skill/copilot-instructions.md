## image-gen-api-skill

> Generate or edit raster images through an OpenAI-compatible Images API, model relay, or CC Switch-managed Codex provider. Use by default instead of the built-in imagegen skill when Codex is running with a custom/non-OpenAI provider, a non-official base_url, IMAGE_API_BASE_URL/OPENAI_BASE_URL, or when the user asks to use CC Switch, a gateway, relay, API, or non-OpenAI image model. Do not use when the task is better handled by SVG/vector/code-native assets.


# Image Gen API Skill

Generates or edits images for the current project through `scripts/image_gen.py`, using an OpenAI-compatible Images API endpoint resolved from environment variables or Codex/CC Switch configuration.

## Default routing

Use this skill as the default image-generation path when any of these are true:

- Codex `~/.codex/config.toml` has `model_provider` set to a custom provider or a provider whose `base_url` is not the official OpenAI API URL.
- The environment contains `IMAGE_API_BASE_URL`, `IMAGE_GEN_API_BASE_URL`, or `OPENAI_BASE_URL`.
- The user asks for CC Switch, a model gateway/relay, API-based image generation, custom `base_url`, custom model, or non-OpenAI image provider.

If Codex is using the built-in official provider and the user did not ask for API/gateway behavior, defer to the system `$imagegen` skill.

## Execution model

This skill has one primary execution mode:

- **API CLI mode:** use `scripts/image_gen.py` for generation, editing, and batch generation. It resolves `base_url`, API key, and model from explicit CLI args, environment variables, or Codex config.

The CLI exposes three subcommands:

- `generate`
- `edit`
- `generate-batch`

Rules:
- Use `scripts/image_gen.py` directly; do not create one-off SDK runners.
- Do not hard-code provider URLs, model names, or secrets in the skill.
- Never print API keys or bearer tokens. Dry-runs may show `base_url`, model, and config source only.
- If the provider does not support `/v1/images/generations` or `/v1/images/edits`, report that capability mismatch and ask the user to switch to an image-capable provider/model.
- For many distinct assets, use `generate-batch`; do not use `n` as a substitute for separate prompts. `n` is for variants of one prompt.

## Config resolution

`scripts/image_gen.py` resolves configuration in this order:

1. Explicit args: `--base-url`, `--model`, `--api-key-env`, `--provider`, `--codex-config`.
2. Image-specific environment variables: `IMAGE_API_BASE_URL`, `IMAGE_API_KEY`, `IMAGE_API_MODEL`, plus `IMAGE_GEN_API_*` aliases.
3. OpenAI-compatible environment variables: `OPENAI_BASE_URL`, `OPENAI_API_KEY`, `OPENAI_IMAGE_MODEL`.
4. Codex config: `${CODEX_HOME:-~/.codex}/config.toml`, including CC Switch-managed `[model_providers.<name>]` values such as `base_url` and `experimental_bearer_token`.

If a Codex provider `base_url` is a bare host such as `https://gateway.example.com`, the CLI normalizes it to `https://gateway.example.com/v1` for image API calls. Pass `--base-url` to override this exactly.

The image model comes from `--model`, `IMAGE_API_MODEL`/`IMAGE_GEN_API_MODEL`/`OPENAI_IMAGE_MODEL`, or the built-in default `gpt-image-2`. The CLI does not use the current Codex chat model as the default image model.

Save-path policy:
- Default one-off output path: `output/image-gen-api/output.png`.
- If the user names a destination, use that destination.
- If the image is meant for the current project, save the final asset in the workspace and update consuming code or references.
- Do not overwrite an existing asset unless the user explicitly asked for replacement; otherwise create a sibling versioned filename such as `hero-v2.png` or `item-icon-edited.png`.

Shared prompt guidance lives in `references/prompting.md` and `references/sample-prompts.md`.

API docs/resources:
- `references/cli.md`
- `references/image-api.md`
- `references/codex-network.md`
- `scripts/image_gen.py`

Local post-processing helper:
- `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.py`: removes a flat chroma-key background from a generated image and writes a PNG/WebP with alpha. Prefer auto-key sampling, soft matte, and despill for antialiased edges.

## When to use
- Generate a new image (concept art, product shot, cover, website hero)
- Generate a new image using one or more reference images for style, composition, or mood
- Edit an existing image (inpainting, lighting or weather transformations, background replacement, object removal, compositing, transparent background)
- Produce many assets or variants for one task

## When not to use
- Extending or matching an existing SVG/vector icon set, logo system, or illustration library inside the repo
- Creating simple shapes, diagrams, wireframes, or icons that are better produced directly in SVG, HTML/CSS, or canvas
- Making a small project-local asset edit when the source file already exists in an editable native format
- Any task where the user clearly wants deterministic code-native output instead of a generated bitmap

## Decision tree

Think about two separate questions:

1. **Intent:** is this a new image or an edit of an existing image?
2. **Execution strategy:** is this one asset or many assets/variants?

Intent:
- If the user wants to modify an existing image while preserving parts of it, treat the request as **edit**.
- If the user provides images only as references for style, composition, mood, or subject guidance, treat the request as **generate**.
- If the user provides no images, treat the request as **generate**.

API edit semantics:
- API edit mode can accept local image file paths via `--image`.
- The selected provider must support `/v1/images/edits`; many relays only support generation.
- If editing is unsupported, report the provider limitation and suggest switching provider/model or using the built-in `$imagegen` path for that edit.
- For edits, preserve invariants aggressively and save non-destructively by default.

Execution strategy:
- For a single asset or variants of one prompt, use `generate`.
- For many distinct assets, use `generate-batch` with JSONL input.
- Use `edit` only when preserving/modifying existing images is central to the request.

Assume the user wants a new image unless they clearly ask to change an existing one.

## Workflow
1. Decide routing: if the current Codex provider is custom/CC Switch-managed or the user asked for API/gateway behavior, use this skill. Otherwise defer to `$imagegen`.
2. Decide the intent: `generate` or `edit`.
3. Decide whether the output is preview-only or meant to be consumed by the current project.
4. Decide the execution strategy: single asset vs variants vs `generate-batch`.
5. Collect inputs up front: prompt(s), exact text (verbatim), constraints/avoid list, and any input images.
6. For every input image, label its role explicitly:
   - reference image
   - edit target
   - supporting insert/style/compositing input
7. For local edit targets, inspect with `view_image` when visual understanding matters, then pass the path to `scripts/image_gen.py edit`.
8. If the user asked for a photo, illustration, sprite, product image, banner, or other explicitly raster-style asset, use this API workflow rather than substituting SVG/HTML/CSS placeholders. If the request is for an icon, logo, or UI graphic that should match existing repo-native SVG/vector/code assets, prefer editing those directly instead.
9. Augment the prompt based on specificity:
   - If the user's prompt is already specific and detailed, normalize it into a clear spec without adding creative requirements.
   - If the user's prompt is generic, add tasteful augmentation only when it materially improves output quality.
10. Use `scripts/image_gen.py` from this skill. Start with `--dry-run` when checking config, paths, or generated payloads.
11. For transparent-output requests, follow the transparent image guidance below: generate on a flat chroma-key background through the API, then run the installed `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.py` helper and validate the alpha result. Use native transparency only when the provider/model is known to support it or the user explicitly requests it.
12. Inspect outputs and validate: subject, style, composition, text accuracy, and invariants/avoid items.
13. Iterate with a single targeted change, then re-check.
14. For preview-only work, render the generated image inline and leave it at the selected output path.
15. For project-bound work, save the artifact in the workspace and update any consuming code or references.
16. For batches or multi-asset requests, persist every requested deliverable final in the workspace unless the user explicitly asked to keep outputs preview-only. Discarded variants do not need to be kept unless requested.
17. Always report the final saved path(s), final prompt or prompt set, model, and non-secret API config source. For each final image, include an inline Markdown image preview using the absolute filesystem path.

## Transparent image requests

Transparent-image requests use the API first. Because many relays do not expose true transparent-background controls, create a removable chroma-key source image and then convert the key color to alpha locally.

Default sequence:
1. Use `scripts/image_gen.py generate` to generate the requested subject on a perfectly flat solid chroma-key background.
2. Choose a key color that is unlikely to appear in the subject: default `#00ff00`, use `#ff00ff` for green subjects, and avoid `#0000ff` for blue subjects.
3. Save the selected source image in the workspace or `tmp/image-gen-api/`.
4. Run the installed helper path, not a project-relative script path:
   ```bash
   python "${CODEX_HOME:-$HOME/.codex}/skills/.system/imagegen/scripts/remove_chroma_key.py" \
     --input <source> \
     --out <final.png> \
     --auto-key border \
     --soft-matte \
     --transparent-threshold 12 \
     --opaque-threshold 220 \
     --despill
   ```
5. Validate that the output has an alpha channel, transparent corners, plausible subject coverage, and no obvious key-color fringe. If a thin fringe remains, retry once with `--edge-contract 1`; use `--edge-feather 0.25` only when the edge is visibly stair-stepped and the subject is not shiny or reflective.
6. Save the final alpha PNG/WebP in the project if the asset is project-bound.

Prompt transparent requests like this:

```text
Create the requested subject on a perfectly flat solid #00ff00 chroma-key background for background removal.
The background must be one uniform color with no shadows, gradients, texture, reflections, floor plane, or lighting variation.
Keep the subject fully separated from the background with crisp edges and generous padding.
Do not use #00ff00 anywhere in the subject.
No cast shadow, no contact shadow, no reflection, no watermark, and no text unless explicitly requested.
```

Do not automatically assume native transparency support. Ask or verify provider support before passing `--background transparent --output-format png`, especially for complex subjects: hair, fur, smoke, glass, liquids, translucent materials, reflective objects, soft shadows, realistic product grounding, or subject colors that conflict with all practical key colors.

Use a concise confirmation like:

```text
This likely needs true native transparency. The default API path uses a chroma-key background plus local removal, while native transparency depends on the current gateway/model. Should I try native transparency through the configured provider, or use chroma-key removal?
```

## Prompt augmentation

Reformat user prompts into a structured, production-oriented spec. Make the user's goal clearer and more actionable, but do not blindly add detail.

Treat this as prompt-shaping guidance, not a closed schema. Use only the lines that help, and add a short extra labeled line when it materially improves clarity.

### Specificity policy

Use the user's prompt specificity to decide how much augmentation is appropriate:

- If the prompt is already specific and detailed, preserve that specificity and only normalize/structure it.
- If the prompt is generic, you may add tasteful augmentation when it will materially improve the result.

Allowed augmentations:
- composition or framing hints
- polish level or intended-use hints
- practical layout guidance
- reasonable scene concreteness that supports the stated request

Not allowed augmentations:
- extra characters or objects that are not implied by the request
- brand names, slogans, palettes, or narrative beats that are not implied
- arbitrary side-specific placement unless the surrounding layout supports it

## Use-case taxonomy (exact slugs)

Classify each request into one of these buckets and keep the slug consistent across prompts and references.

Generate:
- photorealistic-natural — candid/editorial lifestyle scenes with real texture and natural lighting.
- product-mockup — product/packaging shots, catalog imagery, merch concepts.
- ui-mockup — app/web interface mockups and wireframes; specify the desired fidelity.
- infographic-diagram — diagrams/infographics with structured layout and text.
- scientific-educational — classroom explainers, scientific diagrams, and learning visuals with required labels and accuracy constraints.
- ads-marketing — campaign concepts and ad creatives with audience, brand position, scene, and exact tagline/copy.
- productivity-visual — slide, chart, workflow, and data-heavy business visuals.
- logo-brand — logo/mark exploration, vector-friendly.
- illustration-story — comics, children’s book art, narrative scenes.
- stylized-concept — style-driven concept art, 3D/stylized renders.
- historical-scene — period-accurate/world-knowledge scenes.

Edit:
- text-localization — translate/replace in-image text, preserve layout.
- identity-preserve — try-on, person-in-scene; lock face/body/pose.
- precise-object-edit — remove/replace a specific element (including interior swaps).
- lighting-weather — time-of-day/season/atmosphere changes only.
- background-extraction — transparent background / clean cutout. Use the API chroma-key workflow first for simple opaque subjects; verify provider support before using native transparency.
- style-transfer — apply reference style while changing subject/scene.
- compositing — multi-image insert/merge with matched lighting/perspective.
- sketch-to-render — drawing/line art to photoreal render.

## Shared prompt schema

Use the following labeled spec as shared prompt scaffolding for both top-level modes:

```text
Use case: <taxonomy slug>
Asset type: <where the asset will be used>
Primary request: <user's main prompt>
Input images: <Image 1: role; Image 2: role> (optional)
Scene/backdrop: <environment>
Subject: <main subject>
Style/medium: <photo/illustration/3D/etc>
Composition/framing: <wide/close/top-down; placement>
Lighting/mood: <lighting + mood>
Color palette: <palette notes>
Materials/textures: <surface details>
Text (verbatim): "<exact text>"
Constraints: <must keep/must avoid>
Avoid: <negative constraints>
```

Notes:
- `Asset type` and `Input images` are prompt scaffolding, not dedicated CLI flags.
- `Scene/backdrop` refers to the visual setting. It is not the same as the API `background` parameter, which controls output transparency behavior when the provider supports it.
- Execution notes such as `Quality:`, `Input fidelity:`, masks, output format, and output paths are CLI/API controls. Use only the controls supported by the selected provider.

Augmentation rules:
- Keep it short.
- Add only the details needed to improve the prompt materially.
- For edits, explicitly list invariants (`change only X; keep Y unchanged`).
- If any critical detail is missing and blocks success, ask a question; otherwise proceed.

## Examples

### Generation example (hero image)
```text
Use case: product-mockup
Asset type: landing page hero
Primary request: a minimal hero image of a ceramic coffee mug
Style/medium: clean product photography
Composition/framing: wide composition with usable negative space for page copy if needed
Lighting/mood: soft studio lighting
Constraints: no logos, no text, no watermark
```

### Edit example (invariants)
```text
Use case: precise-object-edit
Asset type: product photo background replacement
Primary request: replace only the background with a warm sunset gradient
Constraints: change only the background; keep the product and its edges unchanged; no text; no watermark
```

## Prompting best practices
- Structure prompt as scene/backdrop -> subject -> details -> constraints.
- Include intended use (ad, UI mock, infographic) to set the mode and polish level.
- Use camera/composition language for photorealism.
- Only use SVG/vector stand-ins when the user explicitly asked for vector output or a non-image placeholder.
- Quote exact text and specify typography + placement.
- For tricky words, spell them letter-by-letter and require verbatim rendering.
- For multi-image inputs, reference images by index and describe how they should be used.
- For edits, repeat invariants every iteration to reduce drift.
- Iterate with single-change follow-ups.
- If the prompt is generic, add only the extra detail that will materially help.
- If the prompt is already detailed, normalize it instead of expanding it.
- See `references/cli.md` and `references/image-api.md` for model, provider, `quality`, `input_fidelity`, masks, output format, and output-path guidance.
- For transparent images, use the API chroma-key workflow unless native transparency is known to be supported by the selected provider/model.

More principles shared by both modes: `references/prompting.md`.
Copy/paste specs shared by both modes: `references/sample-prompts.md`.

## Guidance by asset type
Asset-type templates (website assets, game assets, wireframes, logo) are consolidated in `references/sample-prompts.md`.

## Provider and Model Guidance

The CLI is OpenAI-compatible, but provider behavior varies.

- If `IMAGE_API_MODEL` or `OPENAI_IMAGE_MODEL` is set, use that model.
- If no image-specific model is set, the CLI defaults to `gpt-image-2` because the current Codex chat model is usually not image-capable.
- By default the CLI does not send a `quality` parameter; output quality is the provider/model default. Add `--quality` only when the selected provider supports it and the task needs it.
- Prefer omitting optional controls first. Add `--size`, `--quality`, `--background`, `--output-format`, and `--input-fidelity` only when the provider/model supports them or the user needs the control.
- For GPT Image models, `gpt-image-2` supports flexible `WIDTHxHEIGHT` sizes subject to its constraints; older GPT Image models typically use `1024x1024`, `1536x1024`, `1024x1536`, or `auto`.
- For non-OpenAI models, treat size/quality/background support as provider-specific and retry without unsupported optional parameters when that does not violate the user's request.

Common sizes:
- `1024x1024` square
- `1536x1024` landscape
- `1024x1536` portrait
- `2048x2048` 2K square
- `2048x1152` 2K landscape

## CLI Mode

### Temp and output conventions
These conventions apply to `scripts/image_gen.py`.
- Use `tmp/image-gen-api/` for intermediate files (for example JSONL batches); delete them when done.
- Write final artifacts under `output/image-gen-api/`.
- Use `--out` or `--out-dir` to control output paths; keep filenames stable and descriptive.

### Dependencies
Prefer `uv` for dependency management in this repo.

Required Python package:
```bash
uv pip install openai
```

Required for local chroma-key removal and optional downscaling:
```bash
uv pip install pillow
```

Portability note:
- If you are using the installed skill outside this repo, install dependencies into that environment with its package manager.
- In uv-managed environments, `uv pip install ...` remains the preferred path.

### Environment
- A live API call needs an API key from `IMAGE_API_KEY`, `IMAGE_GEN_API_KEY`, `OPENAI_API_KEY`, `--api-key-env`, or Codex provider config.
- A live API call needs a compatible image endpoint from `IMAGE_API_BASE_URL`, `IMAGE_GEN_API_BASE_URL`, `OPENAI_BASE_URL`, `--base-url`, or Codex provider config.
- Never ask the user to paste the full key in chat. Ask them to set it locally and confirm when ready.

If the key is missing, give the user these steps:
1. Configure CC Switch/Codex with an image-capable provider, or set `IMAGE_API_KEY`/`OPENAI_API_KEY` locally.
2. Set `IMAGE_API_MODEL` only if the gateway's image model should differ from the default `gpt-image-2`.
3. Run a `--dry-run` first to confirm model, base URL, and output paths without making a network call.

If installation is not possible in this environment, tell the user which dependency is missing and how to install it into their active environment.

### Script-mode notes
- CLI commands + examples: `references/cli.md`
- API parameter quick reference: `references/image-api.md`
- Network approvals / sandbox settings for CLI mode: `references/codex-network.md`

## Reference map
- `references/prompting.md`: shared prompting principles.
- `references/sample-prompts.md`: shared copy/paste prompt recipes.
- `references/cli.md`: CLI usage via `scripts/image_gen.py`.
- `references/image-api.md`: OpenAI-compatible image API parameter reference.
- `references/codex-network.md`: network/sandbox troubleshooting for CLI mode.
- `scripts/image_gen.py`: CLI implementation for generate, edit, and generate-batch.
- `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.py`: local post-processing helper for chroma-key transparent-image requests.

---
> Source: [SonnetZ/image-gen-api-skill](https://github.com/SonnetZ/image-gen-api-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
