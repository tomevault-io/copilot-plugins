## deswik-claude-project

> You are working on a Python SDK that reads, edits, and generates Deswik

# Deswik Process Map SDK — working notes for Claude

You are working on a Python SDK that reads, edits, and generates Deswik
process map files (`*.ddf`). **Read `KNOWLEDGE.md` first** — it is the
self-contained knowledge base (file format, Tag grammar, command library,
embedded-macro recipe, DLL inspection, CLI usage). Don't re-derive what's
already there.

## Fast facts
- Maps live in `C:\ProgramData\Deswik\Workflows` (hidden). Deswik install:
  `C:\Program Files\Deswik\Deswik.Suite 2024.2`.
- `.ddf` = `01 32|33` header + two varint-prefixed UTF-8 XML blobs + opaque
  .NET tail. Don't re-encode the XML.
- Node `<Tag>` = commands (`Name:payload` + `Use=`/`Description=`) then node
  props, joined by `**||**`. Payloads are `~~~^^^~~~`-delimited or embedded
  `<NewDataSet>` XML.
- Embedded macros: **Deswik.Graphics.* types only** (VectorDraw is
  unreachable). Create layers with the `CreateLayers` command, not in the
  macro. See `docs/MACRO-RECIPE.md`.

## How to work here
- Inspect a map: `python -m deswik_pm inspect "<file>.ddf"`.
- Generate a shape map: `python -m deswik_pm draw circle|square|triangle|polygon|text [...]`.
- Build in Python: `deswik_pm.ProcessMapBuilder` + `deswik_pm.macros` + typed
  commands from `deswik_pm.commands`.
- After changing the SDK, run `python tests/test_roundtrip.py` and
  `python tests/test_commands.py` — both must print ALL TESTS PASSED.
- To verify a generated map actually runs: copy it into the Workflows folder;
  the **user** opens the Process Map window and clicks the node, then reports
  the result. You cannot drive the Deswik desktop UI.

## Adding a new command (the scaling workflow)
The ~51 unverified commands in `commands/library.py` have *guessed* internal
names. To make one real: have the user build a minimal sample `.ddf` with that
command in Deswik, inspect its payload, save it to `docs/samples/`, promote the
stub to a typed class (mirror `CreateLayers`), and add a round-trip test.
This is how `CreateLayers` and the `0x33` header support were added.

## Inspecting Deswik DLLs
Use `dnfile` to read managed metadata (loading the assembly fails on deps);
silence its warnings with `logging.disable(logging.CRITICAL)`. UI editor
classes have obfuscated field names — capture real payloads instead of reading
them. Example in `KNOWLEDGE.md` §8.

## Conventions
- Don't redistribute `docs/` (Deswik copyrighted) or real `*.ddf` (client data).
- Don't push to GitHub without the user's explicit go-ahead (they verify
  locally first).
- Verify macro APIs against the DLL before writing them; WWB validates calls
  even inside Try/Catch.

---
> Source: [botaoshen/Deswik_Claude_Project](https://github.com/botaoshen/Deswik_Claude_Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
