## openrouter

> This workspace is dedicated to researching, testing, and comparing open-source AI models using OpenRouter. It serves as a centralized hub for tracking model performance, routing strategies, and implementing harnesses for dynamic model usage.

# Project Context: OpenRouter AI Research & Sandbox

This workspace is dedicated to researching, testing, and comparing open-source AI models using OpenRouter. It serves as a centralized hub for tracking model performance, routing strategies, and implementing harnesses for dynamic model usage.

## Rules & Guidelines

1.  **OpenRouter Focus**: All model interactions and documentation should prioritize OpenRouter as the primary gateway for accessing open-source models.
2.  **Open Source First**: Focus primarily on top-tier open-source models (e.g., Llama 3, Mixtral, Qwen, DeepSeek, etc.) and their specific strengths.
3.  **Credential Safety**: Never hardcode API keys. Use `.env` files and ensure they are ignored by version control (though this is a local sandbox, follow best practices).
4.  **Model Precision**: Always use exact OpenRouter model IDs (e.g., `meta-llama/llama-3-70b-instruct`) when discussing or implementing model calls.
6.  **Documentation Maintenance**: Keep `docs/models.md` and `docs/rankings.md` up-to-date with current rankings and new model releases.
7.  **Model Pulse Directive**: Regularly research and update `docs/rankings.md` based on OpenRouter usage statistics, Arena Elo, and SWE-bench performance to ensure recommendations are current.
8.  **Knowledge Base Search Rule**: Whenever the user asks for a model recommendation for a specific task (e.g., "coding", "research", "email"), the agent MUST use `grep_search` or `read_file` on `docs/openrouter_database.md` to identify the highest-ranking model for that category before providing an answer.

6.  **Modular Sandbox**: Ensure sandbox scripts are modular, allowing for easy switching of models and providers.

## Structure

- `docs/`: Knowledge base for models and routing techniques.
- `sandbox/`: Test scripts and benchmarks.
- `.env.example`: Template for required environment variables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/squishier8123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
