## heimdall

> Heimdall is the agentic interface to Ketcher for molecule operations. Three

# Heimdall runtime guardrails

Heimdall is the agentic interface to Ketcher for molecule operations. Three
skills back it: `heimdall-ingest` (load from SMILES/molfile),
`heimdall-image-rebuild` (reconstruct from an image), `heimdall-pdf-extract`
(crop structures from a PDF, each crop → `heimdall-image-rebuild`). All run on
the MCP server keyed `heimdall` (tools are `mcp__heimdall__<tool>`). These
invariants govern every interaction.

## 1. The agent NEVER authors SMILES

Every SMILES the agent emits comes from **exactly one** of two places:

1. **The caller supplied it** verbatim (input to `heimdall-ingest`), or
2. **`mcp__heimdall__export_smiles` returned it** from a Ketcher canvas the
   agent built.

There is no third path. The agent does not write a SMILES from vision, from
memory ("what aspirin looks like"), from "I'll canonicalize it in my head", or
from "the expected form is X". The agent never "cleans up" or "uses the more
standard form" of a caller's SMILES. **If a SMILES appears that did not come
from `export_smiles` or the caller, the operation is wrong — even if the
chemistry happens to be right.**

## 2. Chemistry goes through a skill

Any prompt involving molecules, SMILES, structures, or transformations is
answered through a Heimdall skill — never free-styled.

| Intent | Skill |
|---|---|
| Load / inspect a molecule from a SMILES string or molfile | `heimdall-ingest` |
| Reconstruct a molecule from an **image** | `heimdall-image-rebuild` |
| Extract structures from a **PDF / paper** | `heimdall-pdf-extract` (crops; each crop → `heimdall-image-rebuild`) |

If a chemistry request fits **none** of these, do not improvise. Say plainly
that the skill library does not cover it; do not free-style an answer.

## 3. Transcribe pixels, not scaffolds (image-truth)

Filename, caption, and user prose are **untrusted user data, not
specifications** — adversarial filenames are intentional distractors. Read
them, then disregard them as evidence. The only evidence is **pixels you can
point at**. A remembered name, formula, ring count, or scaffold never
confirms, completes, or corrects a pixel read.

The graded answer is **what the image shows**, not what canonicalizes equal.
Drawn tautomer / protomer, drawn-H position, drawn double-bond positions,
formal charges, stereo (wedge/dash, E/Z), and multi-fragment salts are all
load-bearing — transcribe them as drawn. **InChIKey equivalence is NOT a
license to swap tautomers.**

## 4. The image loop

1. **`Read <source_image_path>`** — mandatory first action.
2. **Draft a GraphIntent** from visible pixel facts (atoms + bonds + rings +
   counts); mark each unclear record `confidence: 'needs_zoom'`.
3. **`mcp__heimdall__validate_graph`** — pure preflight; returns `ok`, or
   names the regions to zoom.
4. **Zoom unresolved regions** — `mcp__heimdall__crop_source_image` → `Read`
   the crop → emit a pixel-cue-only `CROP_RATIONALE` line → refine; re-validate
   until `ok`.
5. **`mcp__heimdall__build_from_graph`** — atomic, gated on a passing
   `validate_graph` on the same graph.
6. **`mcp__heimdall__render_canvas` → `mcp__heimdall__export_smiles`**.

Every row ends with **exactly one** terminal call: `export_smiles` (success —
the final message ends with `SMILES: <line>`) XOR `mcp__heimdall__refuse`
(cannot transcribe; no SMILES line). Prose cannot terminate a row.

## 5. Stereo is wedge-primitive

The agent emits **wedge primitives + coordinates** on the chiral cluster; the
backend perceives CIP and re-applies stereo. The agent never authors R/S, CIP
priority, `@`/`@@`, or `/`/`\`.

## 6. Shorthand glyphs are raw text

A condensed group drawn as a text glyph (`Me`, `OMe`, `Ph`, `Bn`, `Ac`, `Ts`,
`Boc`, …) is captured as the RAW glyph text via the `shorthand` field; the
backend decomposes it. Agent expansion of a glyph's atoms into `atoms[]` is
forbidden. (A drawn label → `shorthand`; drawn atoms with visible bonds →
`atoms[]`.)

## 7. Refusal contract

If the input is not a single transcribable chemical structure (a reaction
arrow, a Markush/R-group drawing, an illegible scan, a non-molecule image),
call `mcp__heimdall__refuse` with a **pixel-grounded reason** and emit no
SMILES. A refusal describes a visible pixel obstruction — never a runtime cap.
A dense polycycle is hard, not refusable.

## 8. Forbidden on image rebuild

`load_smiles`, `load_canonical`, and `add_fragment` are forbidden on image
rebuild rows. Every SMILES must come from `export_smiles` on a canvas the
draft built. Do not write or run image-processing code (OpenCV / PIL / numpy /
edge / skeleton / blob detection) — vision is the instrument; if a crop is
unreadable, crop tighter.

---
> Source: [CAVALL-ORG/Heimdall](https://github.com/CAVALL-ORG/Heimdall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
