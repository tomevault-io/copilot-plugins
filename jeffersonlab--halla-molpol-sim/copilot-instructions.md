## halla-molpol-sim

> C++ code style and conventions for MolPol project


# C++ Code Style Rules

## File Extensions
- Header files must use `.hh` extension (not `.h` or `.hpp`)
- Source files must use `.cc` extension (not `.cpp`)

## Naming Conventions
- All classes must use UpperCamelCase (PascalCase) with "MolPol" prefix
- Member variables must be prefixed with `f` (e.g., `fDetNo`, `fHitColl`, `fPartPos`)
- Access specifiers (`public:`, `private:`) must have colon on the same line

## Include Guards
- Use format: `__MOLPOL[CLASSNAME]_HH` (all uppercase)
- Example: `#ifndef __MOLPOLDETECTOR_HH` / `#define __MOLPOLDETECTOR_HH`
- End guard with: `#endif//__MOLPOL[CLASSNAME]_HH`

## Include Order
1. Local project headers first (e.g., `#include "MolPolDetector.hh"`)
2. GEANT4 headers second (e.g., `#include "G4VSensitiveDetector.hh"`)
3. Standard library headers last (e.g., `#include <vector>`)

## Indentation
- Use tabs (not spaces) for indentation

## Comments
- Single-line comments: `//`
- Multi-line comments: `/* */`
- Documentation comments: `/*! */` for Doxygen-style

## GEANT4 Types
- Always use GEANT4 types: `G4String`, `G4int`, `G4double`, `G4bool`, `G4ThreeVector`, etc.
- Use GEANT4 units system: `G4SystemOfUnits.hh` (e.g., `nanobarn`, `mm`, `cm`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeffersonLab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JeffersonLab)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
