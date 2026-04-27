## genmedia-izumi-agent

> - Always use `uv run` to run Python scripts.

- Always use `uv run` to run Python scripts.
- When installing dependencies, use `pyproject.toml` and `uv sync`, not `requirements.txt`
- Don't use Gemini API key, but always use Vertex for this project.
- When you need to work with ADK / google.adk / Google Agent Development Kit, read the GEMINI_for_ADK.md file for deep-dive instructions.
- You can read source code of Python libraries that I use by running `uv run pip show {PACKAGE_NAME}` to find where it is installed in the project's local venv.
- **Format Before Completion**: Always run `uv run pre-commit run --all-files` and ensure it passes before finishing execution.

---
> Source: [GoogleCloudPlatform/genmedia-izumi-agent](https://github.com/GoogleCloudPlatform/genmedia-izumi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
