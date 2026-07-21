## unity-movefast

> Unity showcase of the Meta XR Interaction SDK used for fast-action, hand-tracked fitness gameplay (punch, chop, block). Shipped to App Lab as the "Move Fast" demo.

# Agent Instructions — Move Fast

Unity showcase of the Meta XR Interaction SDK used for fast-action, hand-tracked fitness gameplay (punch, chop, block). Shipped to App Lab as the "Move Fast" demo.

## Source-of-truth files (read these first, do not duplicate their contents in this file)

For setup, build steps, SDK versions, and project layout, read:

- `README.md` — official setup and the App Lab link
- `ProjectSettings/ProjectVersion.txt` — Unity editor version
- `Packages/manifest.json` — Unity package versions (including the Oculus Integration / Interaction SDK)
- `Assets/Project/` and `Assets/MoveFast/` — gameplay scripts and assets specific to this sample
- `Assets/MoveFast/Scenes/MoveFast.unity` — main scene
- `.gitattributes` — Git LFS configuration (LFS is required)
- `LICENSE.txt` — MIT for most of the project; `Assets/TextMesh Pro` is under Unity's Companion License

## Quest / Horizon-specific notes

- Git LFS is required — run `git lfs install` before cloning, otherwise large assets land as text pointers and the project fails to open cleanly in Unity.
- The Interaction SDK lives under `Packages/com.oculus.integration.interaction` (it ships vendored inside the project, not as a Package Manager dependency). Do not remove or re-import it from the Asset Store without understanding the version pinning.
- The README's prose mentions an older "Oculus SDK v46" — `Packages/manifest.json` is the source of truth for the actual pinned versions; do not propagate the README's version number into new docs.
- Hand tracking is the primary input — the demo expects controllers to be set down and hands to be visible. When testing in the editor without a headset, hand input will not be present.

# Meta Quest tooling

This is a Meta Quest / Horizon OS sample. The bespoke intro above is the source of truth for what this project is and how it's built — use it (and the files it points at) instead of restating facts from memory.

When the user asks anything about Quest device behavior, build / deploy / debug / capture flows, on-device performance, or Horizon OS APIs, reach for these tools instead of generic Unity answers:

- **`hzdb`** — Quest-aware ADB wrapper (device list, install / launch / stop, logs, screenshots, Perfetto traces, on-device docs search). Already wired up as an MCP server via `.mcp.json`, `.vscode/mcp.json`, and `.cursor/mcp.json`. Also runnable directly: `npx -y @meta-quest/hzdb <subcommand>`.
- **Meta Quest Agentic Tools** — the full skill set, including Unity-specific skills: <https://github.com/meta-quest/agentic-tools>. Install per your client (Claude Code: `/plugin install meta-vr@meta-quest`; Gemini CLI: `gemini extensions install https://github.com/meta-quest/agentic-tools`; Cursor / VS Code: install the **Meta Horizon** extension from the Marketplace).

A few behavior expectations:

- **Read this repo's files first.** Before answering anything project-specific, read `README.md` and whichever source-of-truth files the intro above points at. Don't restate their contents in chat — quote or link instead.
- **Use `hzdb` for device-side work.** Anything that touches an attached Quest (install, launch, logs, screenshot, capture, manifest inspection) goes through `hzdb`, not raw `adb`.
- **Check live Horizon OS docs before answering API questions.** `hzdb docs search "..."` queries the live docs; training data on Horizon OS APIs goes stale fast.
- **Don't fabricate SDK / engine versions.** If a version isn't visible in this repo's files, say so rather than guessing.

---
> Source: [oculus-samples/Unity-MoveFast](https://github.com/oculus-samples/Unity-MoveFast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
