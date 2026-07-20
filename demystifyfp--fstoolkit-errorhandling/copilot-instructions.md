## fstoolkit-errorhandling

> - Do not `> nul` or `> NUL` (or any casing variation) on Windows machines to pipe output. This create a file that cannot be deleted and breaks other tooling.

# AGENTS.md

- Be Brief.
- Do not `> nul` or `> NUL` (or any casing variation) on Windows machines to pipe output. This create a file that cannot be deleted and breaks other tooling.
- Do not delete code comments unless that the code is being deleted or the comment is factually incorrect. If you think a comment is wrong, update it to be correct rather than deleting it.
- If you ever encounter something in the project that surprises you, please alert the developer working with you and indicate that this is the case, and ask to add a note to the AGENTS.md file to prevent future agents from having the same issue.

---
> Source: [demystifyfp/FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
