## devo-energy-dome-45-adapter

> - Generate 3D models: OpenSCAD -o models/devo-energy-dome-45-adapter.stl src/devo-energy-dome-45-adapter.scad

# DEVO Energy Dome 45 Adapter - Agent Guidelines

## Project Commands
- Generate 3D models: OpenSCAD -o models/devo-energy-dome-45-adapter.stl src/devo-energy-dome-45-adapter.scad
- Preview in OpenSCAD: OpenSCAD src/devo-energy-dome-45-adapter.scad
- Validate STL: stl-validator models/devo-energy-dome-45-adapter.stl

## Automated Workflow
- STL files are automatically generated via GitHub Actions when `.scad` files change
- Generated file: `models/output.stl`
- Workflow triggers on push/PR to main branch when `src/**/*.scad` files are modified
- Generated STL files are committed back to the repository automatically

## Code Style Guidelines
- Parameter naming: snake_case (e.g., base_diameter, post_diameter)
- Comments: Include parameter explanations and section divisions
- Resolution: Use $fn = 300 for high-quality curved surfaces
- Modularity: Create modules for reusable components (e.g., devo_hat())
- Parameters: Place all adjustable parameters at the top of the file
- Dimensions: Use millimeters (mm) and include units in comments

## 3D Printing Recommendations
- Material: PLA or PETG
- Layer height: 0.16mm or lower
- Infill: 20-30%
- Supports: None required
- Orientation: Flat side down

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmcpheron)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jmcpheron)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
