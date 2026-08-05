## ui-skills

> - Use the parchment neutral scale for text and borders. Use white surfaces for elevated or focused components such as cards, code blocks, dialogs, inputs, and navigation controls when it improves separation and contrast.

# AGENTS.md

## Site Rules

- Use the parchment neutral scale for text and borders. Use white surfaces for elevated or focused components such as cards, code blocks, dialogs, inputs, and navigation controls when it improves separation and contrast.
- Prefer `text-base` for body copy
- Use `text-lg` for section copy and `text-3xl` for major headings.
- Keep headings `font-medium` with `tracking-tight` where appropriate.
- Use `JetBrains Mono` for code, commands, and monospace UI.
- Keep labels and link text in sentence case.
- Prefer light borders, minimal decoration, and generous spacing.

## Skills

- Add skills in `src/data/registry.ts` only.
- Fill `slug`, `user`, `repo`, `rawUrl`, `githubUrl`, `name`, `description`, and `topics`.
- If a new topic is needed, update `TopicSlug`, `topicsBySlug`, `relatedTopicSlugs`, and `primaryDesignTopicSlugs` only when it is a core topic.
- Do not add local `skills/*` files unless the repo actually hosts the skill markdown.

---
> Source: [ibelick/ui-skills](https://github.com/ibelick/ui-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
