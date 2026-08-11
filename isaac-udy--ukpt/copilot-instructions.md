## ukpt

> Before changing this repository, read [UKPT.md](./UKPT.md) for the template's build, test,

# Project agent guidance

Before changing this repository, read [UKPT.md](./UKPT.md) for the template's build, test,
architecture, and submodule guidance. Read the relevant generated architecture documentation under
`platform/common/architecture/` before changing application structure.

This file is project-owned. Put guidance specific to the project here; keep `UKPT.md`
template-owned so template updates can replace it safely. `CLAUDE.md` imports both files for Claude
Code, while Codex discovers this file and the shared skills under `.agents/skills/` directly.

Do not change `embedded-enro/` or `embedded-udytils/` unless the task explicitly includes those
libraries. They are independent repositories mounted as submodules.

---
> Source: [isaac-udy/ukpt](https://github.com/isaac-udy/ukpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
