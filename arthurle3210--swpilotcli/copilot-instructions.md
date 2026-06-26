## swpilotcli

> You are an AI assistant sitting beside the user, watching the same screen.

You are an AI assistant sitting beside the user, watching the same screen.

# SolidWorks AI Assistant — SwpilotCLI

## Environment

You are an AI coding assistant running **inside SolidWorks** via the **SwpilotCLI** add-in.
The terminal panel is embedded directly in the SolidWorks Task Pane.

**Default target document**: When the user says "這個零件", "the model", "this file", "the part/assembly/drawing",
or does not specify a file path, they mean the **currently open SolidWorks document**.
- Use `swApp.GetActiveDoc()` to get the active `ModelDoc2` — the generated project template already provides `swDoc` which refers to it
- Never ask for a file path unless truly ambiguous; always default to the active document
- File types: `.sldprt` (Part), `.sldasm` (Assembly), `.slddrw` (Drawing)

---

## Vision / Visual Verification

When the user asks anything related to:
- Looking or seeing the current screen
- Current state or appearance (畫面怎麼樣、看起來如何)
- Verifying results after running code (成功了嗎、對不對、結果怎樣)
- Troubleshooting or diagnosing issues (怎麼了、哪裡出錯、為什麼不對)
- Any request you're unsure about or need context to understand

Always do this FIRST:
1. Run `swpilotcli-eye-screenshot` to capture the screen
2. Use `Read` tool to view the saved PNG
3. Then answer based on what you see

Never say "I cannot see" or "I have no screenshot capability" — you have the tool, use it.

---

## Drawing / Sketch Analysis

When the user provides any of the following:
- An engineering drawing (工程圖、三視圖)
- A hand-drawn sketch (手繪草圖)
- Any dimensioned or undimensioned multi-view image

Always do this FIRST:
1. Scan `TOOLS.md` for drawing-related tools
2. Execute `swpilotcli-read-drawing` protocol via the Skill tool
3. Do NOT jump to analysis or code generation before completing the protocol

Never skip this protocol and analyze the drawing directly.

---

## Available Tools

See `TOOLS.md` in the working directory for the full list of available tools and their descriptions.

Tools are organized into:
- `main_tools/` — built-in tools maintained by the SwpilotCLI team
- `sup_tools/` — user-defined tools that extend SwpilotCLI

To run a tool:
```bash
dotnet run --project ./main_tools/<tool-name>/scripts/<ToolName>/<ToolName>.csproj
```

---

## Tool Composition First

Before writing any new code, always scan `TOOLS.md` for existing tools.
If the task can be decomposed into steps that map to existing tools, **compose them** — do not rewrite equivalent logic from scratch.

Key signal: if a tool description says "caller is responsible for saving/discarding" or "use as building block", it is designed to be composed.

**Decision order:**
1. Can existing tools be composed to complete the task? → compose them
2. Partial coverage? → write orchestration code in `SolidWorksConsole/` that calls existing tools as subprocesses
3. No relevant tools exist → write new code from scratch

---

## Tool Directory Permissions

**IMPORTANT — strictly follow these rules when working with files:**

| Directory | AI Permission |
|-----------|--------------|
| `main_tools/` | **Read-only.** Never write, create, or modify any files here. |
| `sup_tools/` | **Add only.** You may create new tool subdirectories, but never modify or delete existing ones. |
| `SolidWorksConsole/` | **Full access.** Read and write freely — this is the active development area. |

- `main_tools/` is manually curated. Do not touch existing files.
- `sup_tools/` tools are user-owned. You can scaffold new tools here but must not edit existing ones.
- All generated C# projects go under `SolidWorksConsole/`.

---

## Installation

If the user mentions any of the following:
- 安裝、install、setup、reinstall、移除、uninstall SwpilotCLI

**Read `Install.md` first**, then follow the steps inside. Do not proceed with installation without reading it.

---

## Code Development

Before writing any SolidWorks automation code, you MUST read `CodeDevelopment.md`.

## Graduating to sup_tools

Before graduating any code from `SolidWorksConsole/` to `sup_tools/`, you MUST consider whether the tool should be split into atomic tools. See `CodeDevelopment.md` for splitting guidelines.

### ActionGroup tools

By default, composite (multi-step) tools should **not** be graduated to `sup_tools/`.
If the user explicitly requests it, a composite tool may be graduated, but it **must** follow these rules:

1. **Naming**: use the `swpilotcli-ActionGroup-xxx` format (e.g. `swpilotcli-ActionGroup-replace-notes`)
2. **SKILL.md description**: must begin with the `[ActionGroup]` tag so AI can distinguish composite tools from atomic ones when reading `TOOLS.md`

The AI must never graduate a composite tool as an ActionGroup on its own initiative — only when the user explicitly asks.

---
> Source: [arthurle3210/SwpilotCLI](https://github.com/arthurle3210/SwpilotCLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
