## jvm-skills

> - **Single author.** Thomas Schilling works on this project alone. Use "I", never "we".

# Blog Writing Guidelines

- **Single author.** Thomas Schilling works on this project alone. Use "I", never "we".
- **No self-referencing links.** The blog is hosted on jvmskills.com — don't link back to jvmskills.com from blog posts. Link to the GitHub repo when referencing source code.
- **jvm-skills repo URL.** Link to skills at `https://github.com/jvm-skills/jvm-skills/blob/main/.claude/skills/<name>/SKILL.md`.
- **Draft posts.** Add `draft: true` to frontmatter to exclude a post from the published site.
- **Don't invent.** Every fact, number, motivation, and internal-monologue line must be grounded in what actually happened — the working session, the repo, the logs. No fabricated "I assumed / I was sure / the taste" introspection, no made-up metrics. If you can't verify it, don't write it.

## Writing guidance (shared with tschuehly.de)

- **Voice** — load the [voice skill](../.claude/skills/voice/SKILL.md) before drafting or editing any post prose. It carries an empirical voice profile (reasoning habits, never-do list) extracted from Thomas's real pre-AI-era posts.
- **Anti-slop** — after writing or substantially editing a post, do one cleanup pass with the [stop-slop skill](../.claude/skills/stop-slop/SKILL.md). For posts about to go live, run a cold audit: spawn a subagent that loads stop-slop in Audit Mode (report-only), then apply its findings yourself.
- **Tone & structure** — [WRITING_STYLE.md](./WRITING_STYLE.md) is the source of truth for formatting; [PERSONA.md](./PERSONA.md) for positioning and point of view.
- **Pronoun override.** The voice profile uses "we" for tutorial walk-throughs; this blog does not. The single-author "I" rule above wins where they conflict.

## AI-assisted disclosure

Every post carries an automatic disclosure banner injected by `site/blog-post-template.html` (the `.post-disclaimer` aside) — most posts are drafted by a coding agent from real working sessions, then reviewed and edited by Thomas. Do **not** add a disclosure line inside individual post bodies; the template handles it for all posts.

---
> Source: [jvm-skills/jvm-skills](https://github.com/jvm-skills/jvm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
