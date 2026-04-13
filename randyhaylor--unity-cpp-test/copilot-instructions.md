## unity-cpp-test

> **Working.** C++ native plugin running in Unity Editor (Linux) — makes a live HTTPS call to keygen.sh and displays the response in the UI.

# unity_cpp_test

**Working.** C++ native plugin running in Unity Editor (Linux) — makes a live HTTPS call to keygen.sh and displays the response in the UI.

See [unity-cpp-test-plan.md](./unity-cpp-test-plan.md) for the full plan, references, and build instructions.

## Current Status

- Linux x86_64 and Android arm64 builds done and deployed to Unity Plugins folder
- C# interop working — `[DllImport]` calls native `testKeygenSh()`, result displayed in TextMeshPro
- Unity Editor round-trip verified on Linux
- Windows, macOS, iOS builds not yet done

## Key Constraints

- Unity 2022.3 LTS
- Target platforms: Windows (x86_64), macOS (x86_64 + arm64), Linux (x86_64), Android (arm64)
- keygen.sh API call is the test surface — `testKeygenSh()` callable from C#
- Success criteria: keygen.sh API response body contains structured JSON with account-specific content (not based on HTTP status code)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RandyHaylor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RandyHaylor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
