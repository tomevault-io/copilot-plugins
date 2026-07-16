## tkubica12-github-io

> - Start with `SPEC.md`; it is the repo-level map and names the source-of-truth files.

# Agent guidance

- Start with `SPEC.md`; it is the repo-level map and names the source-of-truth files.
- Root `/` is the new interactive blog. Classic Jekyll is legacy and lives under `/classic/`.
- For interactive article authoring, use `.agents\skills\interactive-article-authoring\SKILL.md`.
- For rendering, generation, design, and accessibility, use `.agents\skills\interactive-article-generator\SKILL.md`.
- For compact summaries, use `.agents\skills\caveman-compressor\SKILL.md`.
- Prefer `python tools\build_site.py --skip-classic` for interactive-only local builds, then `python tools\validate_site.py`.
- Keep `AGENTS.md` tiny; put detailed rules in `SPEC.md` or the relevant skill.

---
> Source: [tkubica12/tkubica12.github.io](https://github.com/tkubica12/tkubica12.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
