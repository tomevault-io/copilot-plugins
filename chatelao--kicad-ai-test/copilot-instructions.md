## kicad-ai-test

> Create a simple 555 circuit with LED blinking at 1Hz with 30% active / 70% dark cycle time as KiCAD project with schematic and PCB.

# Project goal
Create a simple 555 circuit with LED blinking at 1Hz with 30% active / 70% dark cycle time as KiCAD project with schematic and PCB.

# Details
Use Kibot in the GitHub action workflow to run th ERC and DRC after each push on every branch. Validate the workflow locally:
- `/specifications` : Download KiCAD IPC API definition and components datasheets, converted to .md if need.
- `/.github/workflows` : The GitHub CICD path.
- A SPICE `555..model`
- `555.kicad_sch`, `555.kicad_pcb`, `555.kicad_pro`, `555.kicad_prl` : The project files
- `KICAD_IPC_CALLS.md` : Log all IPC calls used to build the files.
- `DECICISON_LOG.md` : Add per decision step three variants and the seleted options.

# Howto
- Start with a SPICE-Model to verify the design.
- Use SKiDL to derive and draw the schematics. Use the standard components from KiCAD and store the schematics as ".kicad_sch" file.
- Use KiCAD to create ".kisch_pcd" from ".kicad_sch", use CLI and IPC API where appropriate. Try to avoid direct text manipulations.
- Verify the ERC and DRC with KiBot and add the same verification to the GitHub workflow.

# See
- SKiDL: https://devbisme.github.io/skidl/
- https://gitlab.com/kicad/code/kicad/-/raw/master/api/CMakeLists.txt?inline=false
- https://gitlab.com/kicad/code/kicad-python/-/tree/eb0dad5901e1ac2b0489c92c6414d2cf8b8d01a8/examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chatelao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
