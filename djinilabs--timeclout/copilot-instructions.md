## i18n

> i18n rules


- When changing UI bits, always make sure that the labels are translated by either using the lingui `<Trans>...</Trans>` macro or the `i18n.t("{label}")` from `import { i18n } from "@lingui/core";`.
- Currently we support english (the default) and european portuguese.
- Use the `pnpm i18n:extract` to extract the labels from the code and then populate the translation in `apps/frontend/src/locales/{language}`.

---
> Source: [djinilabs/timeclout](https://github.com/djinilabs/timeclout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
