## vn-sprite-expression-workflow

> Read README.md and docs/configuration.md before changing the pipeline.

# Workflow contributor guidance

Read README.md and docs/configuration.md before changing the pipeline.
Keep source RGBA and pixels outside reviewed masks unchanged. Never loosen masks simply to make verification pass. Keep cleanup regions separate from feature extraction regions. Image generation is external to the local assembler; do not add credential-dependent services silently.

Use synthetic test inputs for public tests. Do not commit users' images, prompts, local work packages, secrets, dependency folders or local paths. The six explicitly authorized anonymous showcase images in docs/images are an exception for README display only; do not add character or story identities. After code changes run npm test and python run.py --root .test-output. Only claim Photoshop verification after a real --photoshop round trip. Technical checks do not constitute visual approval.

---
> Source: [TongjiAI4E/VN_Sprite_Expression_Workflow](https://github.com/TongjiAI4E/VN_Sprite_Expression_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
