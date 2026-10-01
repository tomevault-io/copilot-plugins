## publish-always-bump

> Desktop Store publish always uses the default auto-bump path


# Desktop publish — ALWAYS bump

Publish the desktop app with the default command only:

```
py -3 release/publish_app.py --notes "…"
```

It bumps patch, builds Setup, commits + pushes, uploads. **Never** pass
`--no-bump`, `--set-version`, `--version`, or `--exe` unless the user
types that flag themselves. A local `__version__` that is already ahead
of the Store is not a reason to skip the bump — let it bump again.

**HARD — run the local EXE before Store publish.** Do not `publish_app.py`
until you have launched the latest local `dist/UEFN-Ducky-*/UEFN-Ducky.exe`
(or the just-built one-dir EXE) and exercised the changed UI yourself.
Source/`py -m` is not a substitute. Finish the feature in that EXE, then
publish. If the EXE is not built yet: `py -3 build/build_exes.py`, run it,
then publish.

---
> Source: [UEFN-Ducky/UEFN-Ducky](https://github.com/UEFN-Ducky/UEFN-Ducky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
