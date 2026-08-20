## minimax-music3-studio

> This is a native Windows desktop studio: Tauri 2 shell, Rust/Axum service and native C++/CUDA engines.

# MiniMax Music3 Studio guidelines

This is a native Windows desktop studio: Tauri 2 shell, Rust/Axum service and native C++/CUDA engines.

- Do not add Python or Node.js to the runtime path.
- Model engines are adapters; no UI or server code may hardcode one model's behaviour.
- Provider choice is capability-level: music, ASR, LLM and cover art can independently be Local or OpenRouter.
- Keep OpenRouter model identifiers in provider configuration, not source code.
- Every local engine must expose install state, capability metadata, cancellation and progress before it reaches the UI.
- Model weights, API keys, media and caches are runtime user data; never commit them.

---
> Source: [timoncool/MiniMax-Music3-Studio](https://github.com/timoncool/MiniMax-Music3-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
