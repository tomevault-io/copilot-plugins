## diagrams

> This repository is an installable agent skill for producing accurate hand-drawn whiteboard diagrams.

# Repository instructions

This repository is an installable agent skill for producing accurate hand-drawn whiteboard diagrams.

## Mission

Turn a user's described process or system into one clear causal image. Correctness, legibility, and a complete unclipped composition outrank decoration.

## Required operating procedure

1. Read `SKILL.md` and `SCALE.md` completely.
2. Treat the user's prompt as the source of truth.
3. If the prompt lacks either the subject or the main relationships, ask: "What should the diagram explain, and what are the main steps or components and how do they connect?"
4. Do not ask optional questions when SCALE defaults are sufficient.
5. Extract a JSON spec matching `references/INTAKE.md`.
6. Run `python scripts/diagram_prompt.py validate <spec.json>`.
7. Run `python scripts/diagram_prompt.py build <spec.json> --output <prompt.txt>`.
8. Generate the image using the compiled prompt.
9. Inspect the actual image against the source spec and the SCALE evaluation checklist.
10. Correct inaccurate text, arrows, joins, loops, or clipping before delivery.

## Repository changes

- Keep `SKILL.md` concise and procedural.
- Put detailed visual rules in `SCALE.md`.
- Put schemas and examples in `references/`.
- Keep scripts dependency-free unless a dependency is essential.
- Add or update tests for script behavior.
- Never weaken the intake gate to generate generic filler from an underspecified request.
- Never add decorative defaults that conflict with SCALE.

## Validation

Run all of the following before publishing:

```bash
python -m unittest discover -s tests -v
python scripts/diagram_prompt.py validate examples/text-to-image-model.json
python scripts/diagram_prompt.py build examples/text-to-image-model.json --output /tmp/diagram-prompt.txt
python /path/to/skill-creator/scripts/quick_validate.py .
```

Open the example image at full size and verify text, arrows, margins, and causal truth manually.

---
> Source: [cneuralnetwork/diagrams](https://github.com/cneuralnetwork/diagrams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
