## kian

> - Use `ScrollArea` component consistently across pages for better scrollbar UX; avoid native `overflow` scroll containers.

- Use `ScrollArea` component consistently across pages for better scrollbar UX; avoid native `overflow` scroll containers.

- Use English for commit messages, keep them concise and clear, follow git message best practices, and always start the subject with one of these prefixes: `fix:`, `feat:`, `chore:`, or `refactor:`. Prefer user-friendly wording that clearly describes the user-visible problem fixed, feature added, or value unlocked, so users can understand it as a Changelog entry; avoid technical jargon unless the change is a purely technical refactor or maintenance task.

- All user-facing text must be internationalized; do not add or modify visible copy without adding corresponding i18n entries and translations.

- When updating README documentation, keep the English and Chinese versions in sync.

- Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions
simple and focused.

- Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability. Don't add docstrings,
comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-
evident.

- Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and
framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.

- Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical
future requirements. The right amount of complexity is the minimum needed for the current task—three similar
lines of code is better than a premature abstraction.

- Avoid backwards-compatibility hacks like renaming unused _vars, re-exporting types, adding // removed
comments for removed code, etc. If you are certain that something is unused, you can delete it completely.

---
> Source: [SandAI-org/kian](https://github.com/SandAI-org/kian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
