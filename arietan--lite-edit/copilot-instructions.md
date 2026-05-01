## local-build

> After any code change to LiteEdit, always build and launch locally so the user can test immediately.


# Local Build & Launch

After every code change (feature, fix, refactor), **always** end by building and launching the app locally so the user can test immediately. Do not wait to be asked.

## Steps

```bash
cd /path/to/lite-edit
swift build -c release 2>&1
cp .build/release/LiteEdit LiteEdit.app/Contents/MacOS/LiteEdit
open LiteEdit.app
```

## Binary Size Gate

The release binary must stay **under 1 MB**. After every build, check:

```bash
ls -lh .build/release/LiteEdit | awk '{print $5}'
```

If the binary exceeds 1 MB:

1. **Stop** — do not copy it into the app bundle or launch it.
2. Investigate what caused the size increase (new dependencies, embedded resources, debug symbols left in release build, etc.).
3. Fix the issue and rebuild before proceeding.

---
> Source: [arietan/lite-edit](https://github.com/arietan/lite-edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
