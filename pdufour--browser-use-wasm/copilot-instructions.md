## llm-intent-parsing

> Voice transcripts and the Goal "Run task" input go through **one** LLM path: the

# LLM task parsing (ShowUI navigation mode)

Voice transcripts and the Goal "Run task" input go through **one** LLM path: the
ShowUI-2B **UI Navigation** mode from the model card (`src/actions/navigation.js`)
— task + screenshot → `{'action', 'value', 'position'}` dicts.

## Rules

- **Zero Regex Parsing:** NEVER use regular expressions or manual string matching to map natural language to actions — no phrase→tool parsers in shipped code. The E2E harness injects **structured tool calls** (`__e2eVoiceTool`); it never parses phrases.
- **Card-verbatim prompts:** The navigation system prompt, action space, and format come from the ShowUI-2B model card. No custom tool-call JSON prompts, no second intent model/prompt.
- **One inference per utterance:** transcript = `Task:`, screenshot = observation. Execute the returned action sequence (`executeNavigationAction`); do not fan out per-action inferences.
- **Async Flow:** Parsing is asynchronous — promise-chain interpretation and execution.
- **Client-Side Only:** Inference happens entirely in the browser wllama worker.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
