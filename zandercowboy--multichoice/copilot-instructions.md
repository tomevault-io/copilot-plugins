## multichoice

> Rules that apply to specific directories or file patterns


# Conditional Rules Example

This rule file demonstrates how to create rules that apply only to specific directories or file patterns.

## Usage

- `globs`: Can target specific directories or file patterns
- This example applies to core packages and feature directories
- Useful for domain-specific conventions

## Core Package Rules

- All services must have corresponding interfaces
- Interfaces go in `interfaces/` directory
- Implementations go in `implementations/` directory
- Use dependency injection for all services

## Feature Module Rules

- Each feature should be self-contained
- Follow the data/domain/presentation structure
- Use barrel exports (`export.dart`) for public API
- Keep feature dependencies minimal

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZanderCowboy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZanderCowboy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
