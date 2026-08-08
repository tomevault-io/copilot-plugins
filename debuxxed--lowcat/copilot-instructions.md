## lowcat

> - For UI work, use the `gpui` and `gpui-component` skills.

## Notes
- For UI work, use the `gpui` and `gpui-component` skills.
- Whenever you add something to UI, check if the module is imported.
- Do not use `git --diff` unless the user explicitly asks for it.
- Do not set `LOWCAT_PROFILE` for normal `cargo run`; perf logs are only for explicit profiling.

## UI Verification Workflow
- If Codex can verify the UI behavior without user assistance, run `cargo run` and complete verification directly.
- If Codex needs user assistance to verify the UI behavior, ask for manual verification and stop; do not start or keep a live app session just for the user to interact with.
- Add targeted debug output only when it is needed for the affected interaction path, and remove it once the behavior is confirmed.
- When asking for manual verification, tell the user exactly what to try and what result or log is expected.
- Use the user's observation plus the live logs to iterate before declaring the UI fix verified.
- Do not treat compile success alone as UI verification.

---
> Source: [debuxxed/lowcat](https://github.com/debuxxed/lowcat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
