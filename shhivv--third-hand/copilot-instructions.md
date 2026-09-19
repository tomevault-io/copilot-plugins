## third-hand

> - Preserve the user's macOS permissions: build and install using `bash Scripts/build.sh --install`.

# Local Third Hand development

- Preserve the user's macOS permissions: build and install using `bash Scripts/build.sh --install`.
- Never use ad-hoc (`codesign --sign -`) signing for the app the user runs. Do not delete or overwrite `.thirdhand-signing-identity`, change bundle ID, or silently switch signing identity.
- The canonical runnable app is the repository-root `Third Hand.app`. Do not launch the `.build` app or old Desktop copies.
- The build script checks signing-identity continuity and preserves the prior app before installation. Do not bypass failed signing checks or modify TCC databases.
- After an update, verify launch and report actual permission status. Do not claim live app control was tested based only on unit tests.

---
> Source: [shhivv/third-hand](https://github.com/shhivv/third-hand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
