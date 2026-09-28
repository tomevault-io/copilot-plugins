## personal-visual-system

> 1. `visual-style-guide.md` is the visual source of truth.

# Working with Personal Visual System

## Authority

1. `visual-style-guide.md` is the visual source of truth.
2. `tokens/` contains generated values from the guide.
3. `examples/` and `style-preview.html` demonstrate the rules.

The guide wins a conflict. Read it completely before creating an artifact, then inspect tokens and only the examples relevant to the medium.

## Workflow

Understand the supplied content and its semantic relationships. Work in this order: semantics → structure → visual hierarchy → polish. Choose flows for sequences, aligned regions for comparisons, figures for evidence, and reading flow for prose. Do not force new content into an example's layout.

Preserve supplied content and meaning. Do not invent claims or copy fictional demo content into a real artifact. Examples are references, not mandatory templates. Public examples must remain generic and contain no employer materials, private paths, internal prompts, or confidential data.

Use the guide's warm-white canvas, restrained green emphasis, and gray blue only for secondary differentiation. Use dark readable text, Lato with documented CJK fallbacks, thin borders, 6–8 px radii, and whitespace before containers. Avoid excessive cards, gradients, neon, glass effects, and heavy shadows. Use responsive web composition and a fixed 16:9 slide canvas with their respective type scales.

## Changes and validation

Edit palette values only in the marked JSON block in the guide. Run `python3 tools/generate_tokens.py` to update JSON, CSS, and the standalone preview's palette. Preserve embedded fonts and their license. Keep both READMEs aligned. Do not introduce frameworks, build tooling, or duplicate documentation without a concrete need.

Run `python3 tools/validate.py`. Then render affected HTML and visually inspect hierarchy, alignment, spacing, clipping, overflow, contrast, image proportions, excessive cards, and whether structure expresses meaning. Check web layouts at 1440, 768, and 390 px, diagram labels/connections, and every slide at 1600 × 900. Check keyboard interaction where applicable and disclose font fallback. Refresh affected README images using actual renders; automated validation cannot establish visual quality or screenshot freshness.

---
> Source: [TSUNGYENTSAI/personal-visual-system](https://github.com/TSUNGYENTSAI/personal-visual-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
