## flake-templates

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A collection of Nix flake templates. Provides development environment templates available via `nix flake init -t`.

## Architecture

- `flake.nix` — Root flake. Registers each template under the `templates` attribute
- `<template-name>/flake.nix` — The actual template. Defines `devShells`

To add a new template:
1. Create `flake.nix` in a `<template-name>/` directory
2. Add a new entry to `templates` in the root `flake.nix`

## Conventions

- Uses `github:NixOS/nixpkgs/nixpkgs-unstable` for nixpkgs
- Target systems: `x86_64-linux`, `aarch64-linux`, `x86_64-darwin`, `aarch64-darwin`
- Uses the `forAllSystems` pattern to generate `devShells` for all target systems

## Commands

```bash
# List available templates
nix flake show

# Initialize a project with a template (e.g., node)
nix flake init -t github:siraken/flake-templates#node

# Evaluate and check the flake
nix flake check
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siraken) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
