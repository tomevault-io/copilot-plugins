## mineclawd

> This is a Minecraft Architectury Mod, which supports NeoForge 1.21.1, Forge/Fabric 1.20.1. It doesn't and won't support Fabric 1.21.1.

This is a Minecraft Architectury Mod, which supports NeoForge 1.21.1, Forge/Fabric 1.20.1. It doesn't and won't support Fabric 1.21.1.

Always respond in English, regardless of the language used in the question.

There're two branches in this repository, 1.21.1 and 1.20.1.

This mod should be required on servers, but optional on clients. When adding new features, ensure they are implemented in a way that degrades gracefully for clients without the mod. When client is not running the mod, they should still be able to connect to the server and play without major issues. For example, degrade GUI interfaces to chat-based interactions, like using clickable chat messages instead of custom buttons, commands instead of GUI, etc.

You can use runClient configurations to test and ensure at least the game won't crash before you give it to me for testing.

## CHANGELOG Writing Rules

- Focus on user-facing features and outcomes, not low-level implementation details.
- For features newly added in the target version, write them as additions (for example: "Added xxx feature") and briefly explain what the feature does.
- Do not list within-version iterative fixes for features that are new in that same version.
- Only include fixes when they are cross-version fixes (issues that existed in a previous released version and are fixed in the new version).
- Prioritize functional value over technical churn.

---
> Source: [Zhou-Shilin/MineClawd](https://github.com/Zhou-Shilin/MineClawd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
