## sterlinghamilton-com

> Core repo conventions (prompts, Bun, commits)


# Project basics

- **Read prompts first**: always start with `prompts/README.md`. If there is a matching workflow prompt in `prompts/`, follow it.
- **Package manager**: use **Bun** (`bun`, `bunx`), not npm/yarn.
- **Verification**: prefer project scripts (e.g. `bun run check`, `bun run test:unit`, `bun run test:e2e`).

## Commit messages

Use: `type(scope): message.`

- lowercase `type` and `scope`
- always include a scope
- end with a period

For detailed commit workflow and examples, see `.cursor/rules/40-git-commits.mdc`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/grok)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/grok)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
