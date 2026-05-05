## llama-cpp-version-lockstep

> Keep llama.cpp versions aligned across converter, gguf-py hints, and installers


# llama.cpp Version Lockstep

When syncing or changing the bundled `llama.cpp` converter version, keep these aligned in the same task:

- `internal/convert/bundled_converter.go` `BundledConverterLLamacppRef`
- user-facing `gguf-py` install/download hints that reference a llama.cpp tag
- installer defaults in `scripts/install.sh` and `scripts/install.ps1` for downloaded `llama-server`

Goal: users should get a consistent llama.cpp version for:

- bundled `convert_hf_to_gguf.py`
- matching `gguf-py`
- downloaded `llama-server`

Do not sync only the converter and leave installer defaults on an older tag.

If an exact mirrored binary tag is unavailable, either mirror it as part of the task or explicitly choose the fallback tag and update all three surfaces together in the same change.

---
> Source: [OpenCSGs/csghub-lite](https://github.com/OpenCSGs/csghub-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
