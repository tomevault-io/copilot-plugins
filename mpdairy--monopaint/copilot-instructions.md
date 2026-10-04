## monopaint

> When the user's tablet is connected over ADB, finish app changes by building,

# Development workflow

When the user's tablet is connected over ADB, finish app changes by building,
verifying, and installing the updated regular app on that tablet. The user has
authorized this as the default workflow; do not ask again or stop at an APK link.
Use an in-place update (`adb install -r`) that preserves the app's drawings and
settings. An isolated test install does not replace updating the regular app.

Treat updating as a complete save, install, and resume workflow. Before replacing
the APK, let the app finish its active stroke and save its current drawing/page
and settings; record the open drawing/page when practical. Android may stop the
app during package replacement, so immediately reopen its regular launcher
activity after a successful install and verify it returns to the previous
drawing/page. Do not leave the app closed or claim its session was restored
without checking. Preserve existing app data; never uninstall or clear data as
part of a routine update.

---
> Source: [mpdairy/monopaint](https://github.com/mpdairy/monopaint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
