## obsidian-quiz-blocks

> Here is the source code for Obsidian plugin, written in svelte 5 and typescript.

Here is the source code for Obsidian plugin, written in svelte 5 and typescript.
The plugin is about rendering markdown code blocks tagged with `quiz` into interactive quizzes.

- `/src/main.ts` registers the plugin, adds a few obsidian commands, and registers `MarkdownCodeBlockProcessor`
- `/src/schemas.ts` describes the quiz types
- `/src/renderer.ts` is what parses the quiz source block (in YAML) and renders it
- `/src/ui/QuizRenderer.svelte` is a wrapper element which renders concrete interactive quiz
- `/src/ui/Quiz*.svelte` are components per each quiz type

---
> Source: [xamgore/obsidian-quiz-blocks](https://github.com/xamgore/obsidian-quiz-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
