## tooler

> - **Explicit Execution**: Tools must only be executed via explicit user action. Do not implement automated execution features like shell completion generation or post-install hooks.

# Tooler Agent Rules

## Security & Execution

- **Explicit Execution**: Tools must only be executed via explicit user action. Do not implement automated execution features like shell completion generation or post-install hooks.
- **Validation**: Always verify binary presence and executability before every run.

## Testing & Isolation

- **Mandatory Sandboxing**: Utilize all supported isolation environment variables (e.g., `TOOLER_CONFIG`, `TOOLER_DATA_DIR`) for testing.
- **Safety**: Automated tests must never modify the user's production configuration or data.

## Architecture

- **State Hierarchy**: Respect the established hierarchy of state: `config.json` -> Local Storage -> Remote Forge.

## Version Control Standards

- **Semantic Commits**: All commit messages must follow the Conventional Commits specification (e.g., `feat:`, `fix:`, `chore:`, `refactor:`, etc.).
- **Semver**: Use Semantic Versioning (vX.Y.Z) for all tags and version bumps in `Cargo.toml`.
- **Pre-Push Hooks**: The pre-push hook validates commit messages and runs tests. Warnings indicate potential issues that should be addressed. Use `--no-verify` only when necessary (e.g., pushing tags with legacy commits), not to bypass validation warnings.

## Documentation Standards

- **No Time-Based Commitments**: Do not use time-based language like "This Week", "Next Week", or specific dates in documentation. Time is arbitrary and creates overly binding constraints. Use priority levels (Critical/High/Medium/Low) instead.
- **Planning Files**: Markdown files for planning (task_plan.md, findings.md, progress.md, etc.) are local working documents and should not be committed to the repository. They are ignored by .gitignore for this reason.

---
> Source: [morgaesis/tooler](https://github.com/morgaesis/tooler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
