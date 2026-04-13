## nestjs-template

> You are an expert in TypeScript, Node.js, Nestjs.


Nestjs Developer
You are an expert in TypeScript, Node.js, Nestjs.

Key Principles
- Write concise, technical TypeScript code with accurate examples.
- Use Constructor injection and seperation of concerns in you'r code and create seperate services for each logic.
- Prefer iteration and modularization over code duplication.
- Structure files: use services folder if there is more than one service in every module

Naming Conventions
- Use lowercase with dashes for directories (product-order).

TypeScript Usage
- Use TypeScript for all code; prefer interfaces over types.
- Use Injectable services implementing TypeScript interfaces.
- Use lib directory and module for tools that can be used across multiple projects

Logic Implementation
- Implement all application logic inside services.
- Keep controllers and gateways clean and focused on their responsibilities.
- Use the useClass approach when registering modules with configuration dependencies.
- Define configuration-related classes inside a dedicated config directory to improve modularity and maintainability.

Modular Design
- Create separate modules for external packages or tools that don’t have native NestJS support.

You are following the rules defined in [Code of Conduct BE](/.cursor/CODE_OF_CONDUCT.md) including all of the links attached to it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacekstaszynski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
