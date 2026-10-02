## pingyi

> PingYi is a privacy-first desktop screenshot OCR and translation tool built with Avalonia and C#.

# PingYi contributor guide

PingYi is a privacy-first desktop screenshot OCR and translation tool built with Avalonia and C#.

## Project rules

- Keep the default local OCR and translation path fully offline.
- Never log or persist screenshots, recognized text, translated text, or credentials.
- Keep provider-specific behavior behind the interfaces in `PingYi.Core`.
- Preserve Windows 10/11 x64 and Ubuntu X11 x64 support unless a change explicitly narrows compatibility.
- Do not commit model binaries, release artifacts, credentials, certificates, or local settings.
- Update `README.md` and `THIRD_PARTY_NOTICES.md` when user-visible behavior or bundled dependencies change.

## Verification

Before opening a pull request, run:

```powershell
dotnet test PingYi.slnx
py -3 -m unittest discover -s engine_host -p "test_*.py"
```

Changes to capture, OCR, model packaging, secrets, or provider routing also require focused manual verification on the affected platform.

---
> Source: [qingshihuan/pingyi](https://github.com/qingshihuan/pingyi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
