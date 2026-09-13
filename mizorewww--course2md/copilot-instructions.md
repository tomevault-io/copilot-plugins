## course2md

> Before designing, implementing, or reviewing any desktop UI, read and follow

# Project instructions

## Desktop design

Before designing, implementing, or reviewing any desktop UI, read and follow
[course2md-design](.agents/skills/course2md-design/SKILL.md).
It is the current design authority, including when older mockups or design documents disagree.
Keep colors, controls, icons and motion in the shared design primitives; do not create a new visual dialect in a page.

For a whole-product UI change, use subagents to independently review every affected page and its loading, empty, failure and completion states. Record concrete findings and their resolution. Verify the running native application, not just source or HTML mockups.

The user has excluded specialist accessibility audits from this redesign. Preserve ordinary keyboard behavior and existing system preferences without expanding the task into an accessibility project.

Keep changes reviewable and make atomic Git commits by concern. Do not change release tags or replace published assets as part of an ordinary UI change.

---
> Source: [mizorewww/course2md](https://github.com/mizorewww/course2md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
