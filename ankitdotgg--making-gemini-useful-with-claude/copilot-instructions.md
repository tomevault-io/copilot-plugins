## making-gemini-useful-with-claude

> Delegate a task to Gemini using the gemini_prompt MCP tool.

Delegate a task to Gemini using the gemini_prompt MCP tool.

## Instructions

1. Parse the user's request from $ARGUMENTS. If no arguments, ask what they want Gemini to do.

2. Pick the model:
   - `flash` (default) — summaries, research, explanations
   - `pro` — code review, complex analysis, multi-file reasoning
   - `flash-lite` — quick facts, simple lookups

3. If the user references files or folders:
   - Set `working_directory` to the parent folder (absolute path)
   - Pass file paths via the `files` parameter
   - **NEVER read file content yourself** — pass PATHS to Gemini

4. Call `gemini_prompt` with:
   - `prompt`: Clear instruction for Gemini
   - `model`: Chosen alias
   - `working_directory`: Absolute path (if files involved)
   - `files`: List of relative paths (if applicable)
   - `thinking_level`: Set to `high` only for deep analysis requests

5. Present Gemini's response. Add your own commentary only if asked to synthesize.

## Examples

```
/gemini Summarize the README at ~/my-project
/gemini Review ~/src for security issues
/gemini What is UASP support on SSD enclosures
```

<!-- github.com/ankitdotgg/making-gemini-useful-with-claude -->

---
> Source: [ankitdotgg/making-gemini-useful-with-claude](https://github.com/ankitdotgg/making-gemini-useful-with-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
