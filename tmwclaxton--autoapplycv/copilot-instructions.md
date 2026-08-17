## extension-build

> Rebuild the browser extension after any extension source change


# Extension Build

When you change anything under `extension/` or `scripts/build-extension.mjs`, run the extension build before finishing:

```bash
npm run build:extension
```

This updates `extension/dist/` and the downloadable zips in `public/extension/`. Do this by default without waiting for the user to ask.

If the build fails, fix the issue and rerun until it succeeds. Mention in your summary if the build could not be run and the user should run it locally.

After substantive extension changes, bump `extension/manifest.json` version when other extension work in the same task already bumped it or the change is user-visible.

---
> Source: [tmwclaxton/autoapplycv](https://github.com/tmwclaxton/autoapplycv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
