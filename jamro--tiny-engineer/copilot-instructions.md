## compile-after-changes

> After significant code changes, build with pio run to confirm the firmware compiles.


# Compile after significant changes

After any significant code change, run `pio run` from the project root and fix failures before considering the work done.

Significant: logic, types, APIs, headers, firmware source (`src/`, `include/`, `lib/`), or `platformio.ini`.

Skip: comments-only, docs-only, or tiny formatting.

Do not flash (`pio run -t upload`) unless asked.

---
> Source: [jamro/tiny-engineer](https://github.com/jamro/tiny-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
