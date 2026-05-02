## realtime-voice-component

> - When changing package behavior, examples, or docs, reconcile `README.md`, `LLM.txt`, `docs/`, and `examples/` in the same change so they do not drift or contradict each other.

# Repository Instructions

## Documentation Consistency

- When changing package behavior, examples, or docs, reconcile `README.md`, `LLM.txt`, `docs/`, and `examples/` in the same change so they do not drift or contradict each other.
- Before finishing a docs update, check that public API names, option names, defaults, and examples match the implementation.

## Commits

- Use Conventional Commits for all commit messages.
- Format: `type(scope): summary`
- Example types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
- Keep the summary short and imperative.
- Use `docs:` for documentation-only changes.

Examples:

- `feat(widget): add snapToCorners support`
- `fix(auth): handle flattened token responses`
- `docs(readme): clarify ghost cursor usage`

---
> Source: [openai/realtime-voice-component](https://github.com/openai/realtime-voice-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
