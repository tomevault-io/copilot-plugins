## claude-banana

> Claude Banana is an AI image prompt engineering agent. It helps users craft optimized prompts for image generation models — primarily Nano Banana Pro (Gemini image models) but compatible with any image generator.

# Claude Banana

Claude Banana is an AI image prompt engineering agent. It helps users craft optimized prompts for image generation models — primarily Nano Banana Pro (Gemini image models) but compatible with any image generator.

## How It Works

When a user provides a prompt idea, image reference, or existing prompt to refine, the **prompt-architect** agent:

1. **Intake** — Classifies intent (new prompt, refinement, image reference, platform adaptation)
2. **Interview** — Asks 2-4 targeted clarifying questions (mood, domain, must-have elements)
3. **Compose** — Builds an optimized prompt using the 7-component formula with domain-specific weights
4. **Present** — Outputs a copyable master prompt with rationale and variation suggestions
5. **Iterate** — Accepts adjustments without starting from scratch

## Using the Agent

Start Claude Code in this project directory. The prompt-architect agent and knowledge base are automatically available. Just describe what image you want to create:

```
"Help me create a prompt for a moody portrait of a jazz musician"
```

## File Layout

```
knowledge/              — The brain: prompt engineering reference material
  prompt-formula.md     — 7-component formula with weights and construction template
  prompt-engineering.md — Master reference: specificity, detail, anchoring, composition, lighting
  domain-modes.md       — 9 domain profiles with weight adjustments and routing
  techniques-catalog.md — 70+ techniques organized by category
  anti-patterns.md      — Banned keywords, 10 critical mistakes to avoid
  model-notes.md        — Model routing, capabilities, limitations, pricing
  safety-and-text.md    — Text rendering rules, safety filter navigation
  platform-adaptation.md — Converting prompts from Midjourney/DALL-E/SD
  post-processing.md    — ImageMagick/FFmpeg post-processing recipes

templates/examples/     — 25 parameterized prompt templates covering all domains
presets/                — Brand/style preset JSON files
scripts/                — Python utilities for generation, editing, batch processing
```

## Critical Constraints

- **Never pass raw user text directly to the API.** Always enhance through the 7-component formula.
- **Narrative prose, not keyword lists.** Gemini responds to descriptive sentences, not comma-separated tags.
- **No banned keywords**: Never use "4K", "8K", "masterpiece", "highly detailed", "ultra detailed", "trending on ArtStation", "hyperrealistic", "photorealistic", "best quality", "award-winning".
- **No negative prompts**: Gemini has no negative prompt parameter. Rephrase exclusions as positive descriptions.
- **Text rendering limit**: 25 characters max for reliable text in images.
- **`imageSize` requires UPPERCASE**: "1K", "2K", "4K" — not lowercase.
- **One image per API call**: Batching is done through sequential calls.
- **Aspect ratio first**: Always set aspect ratio before generating.

## Dependencies

- Python 3.6+ (scripts use stdlib fallback for compatibility)
- Optional: `google-generativeai` for API scripts
- Optional: ImageMagick 7/6, FFmpeg for post-processing

---
> Source: [Hainrixz/claude-banana](https://github.com/Hainrixz/claude-banana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
