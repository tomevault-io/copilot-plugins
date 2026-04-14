## ontocast

> Use `uv run` for running Python code instead of raw python3


# Running Python code
- **Do not** run `python3 -c` directly.
- Always invoke Python code using `uv run python <script>` or `uv run <module>` when executing project scripts.
- Use the `uv` environment so that dependencies from `uv.lock` are respected.
- This applies to both interactive code execution and testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/growgraph)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/growgraph)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
