## app

> If you are using a numerical constant, either #define it or use the appropriate calculation to derive it (e.g. strlen(some contant)-2 instead of just writing 7)

If you are using a numerical constant, either #define it or use the appropriate calculation to derive it (e.g. strlen(some contant)-2 instead of just writing 7)

When adding logging instructions, use the project-specific ESP_LOG_WEB function

Avoid JSON processing, as it is very memory intensive; try to find what you need with string functions instead

Don't allocate buffers - use the get_shared_buffer function instead (and remember to later release)

---
> Source: [ArtLogicIKE/frixos](https://github.com/ArtLogicIKE/frixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
