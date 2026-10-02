## seerrng

> These interaction rules are standard for all model interfaces used with this repo, including Hermes, Codex CLI, Claude CLI, Kilo CLI, OpenCode, Cursor, and similar agents:

# Agent instructions - seerrng

## Communication style

These interaction rules are standard for all model interfaces used with this repo, including Hermes, Codex CLI, Claude CLI, Kilo CLI, OpenCode, Cursor, and similar agents:

- Never praise questions or validate premises before answers.
- If the user is wrong, say so immediately and directly.
- Do not capitulate under pushback unless new evidence or a stronger argument is provided.
- Do not anchor on numbers or estimates provided by the user. Generate an independent assessment first, then compare.
- Use explicit confidence levels when making claims, recommendations, or estimates: `high`, `moderate`, `low`, or `unknown`.
- Do not add disclaimers.
- Do not give ethics lectures unless explicitly asked.
- Do not use "it is important to consider" style hedges.
- Surface negative conclusions and bad news directly.
- Optimize for accuracy, not approval.
- If you do not know, say so. Never fabricate.

## User interface work

- Before changing user-facing layout, controls, or styling, read
  [`docs/maintainers/ui-style-standard.md`](./docs/maintainers/ui-style-standard.md)
  completely. Follow its shared component, layout, interaction, and review
  requirements; do not substitute an older chat summary for the file.
- Use [`src/styles/globals.css`](./src/styles/globals.css) as the executable
  source for reusable visual styling. Components should refer to semantic CSS
  classes there. Local utility classes may express structural or responsive
  layout, but do not put reusable size, spacing, color, surface, border, or
  effect values in component markup or inline `style` props. Keep data-driven
  geometry only where a static CSS class cannot represent the value.
- Keep the style standard and global CSS synchronized when a shared visual rule
  changes. Do not create a page-specific override or a conflicting component
  size to work around either source.
- Verify shared styling changes in the existing development preview at desktop
  and narrow widths before calling them accepted. Do not create a pull request
  until John explicitly asks for one.
- Do not implement visual styling with inline Tailwind utility strings or
  arbitrary-value Tailwind classes. Tailwind utilities in components are
  limited to structural layout and responsive placement that the visual
  standard explicitly permits.
- Keep these class systems independent: standard application buttons use
  `app-button*`, poster overlays use `poster-control*`, and filter/sort controls
  use `app-filter-button`. Copy approved visual properties into the appropriate
  class family; never make one family borrow another family's class.
- When a new reusable visual treatment is needed, add or extend a semantic
  class in the shared stylesheet, document it in the visual standard, and
  update the style-contract validator before using it in components.
- Before finalizing UI work, run `pnpm ui-style:check` and
  `pnpm current-batch:check`. A successful build does not replace visual review.

## Release-note contract

Do not let user-facing changes reach a release without a user-facing note.
For every feature, bug fix, security change, operational behavior change, or
user-facing documentation change, add one new validated fragment under
`release-notes/` using the format documented in
[`release-notes/README.md`](./release-notes/README.md). Write what changes for
the person running or using SeerrNG, not only the implementation detail.
Every fragment must capture its audience, product area, required action (or
`none`), and breaking-change status so the release can be understood without
reading the commit history. Preview the resulting text with
`pnpm release-notes:preview --base <base> --head <head>` before finalizing work.

If the work is genuinely internal-only, explicitly mark the pull request
`release-note: none`. The pull-request template and CI enforce this choice;
never silently omit it. Release-note fragments are append-only, so add a new
file rather than changing a fragment that has already shipped.

Before declaring a release complete, verify that the generated notes appear in
the GitHub release body and the Discord announcement. The release workflow
assembles curated fragments first and includes the git-cliff technical history
afterward.

Keep `CHANGELOG.md` append-only at the release-section level: the tag workflow
prepends the new section without replacing audited history. When changing
release history or tag preparation, run
`node scripts/check-changelog-tags.mjs`; it must cover every existing `v3.*`
tag. The historical audit and the deliberate absence of `v3.2.6` are documented
in [`docs/maintainers/release-history-audit.md`](./docs/maintainers/release-history-audit.md).

---
> Source: [YunoHost-Apps/seerrng](https://github.com/YunoHost-Apps/seerrng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
