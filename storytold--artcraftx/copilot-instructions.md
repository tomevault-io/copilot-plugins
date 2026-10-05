## artcraftx

> This file provides guidance for Claude Code when working with the ArtCraftX monorepo.

# CLAUDE.md

This file provides guidance for Claude Code when working with the ArtCraftX monorepo.

## Project Overview

ArtCraftX is a pared-down desktop application for generating AI image and video. It is written in
Rust (Tauri) and TypeScript (React) and contains desktop and frontend components.

## Project Structure

```
artcraftx/
├── crates/                              # Rust workspace
│   ├── api_clients/                     # HTTP clients for calling internal and 3rd party services
│   ├── artcraftx/                       # (Important) ArtCraftX, the Tauri desktop app
│   ├── core_types/                      # Highest-level shared types: GenerationSource, CredentialId, ...
│   ├── lib/                             # Various utility libraries
│   ├── router/                          # Provider routing for generation requests (Artcraft, Fal, ...)
│   ├── schema/                          # Data definition layer
│   │   ├── local_files_database/        # Separate SQLite DB: content-hash index of local files (thumbnails, dedup)
│   │   ├── sqlite_database/             # Desktop app's "tasks" SQLite database: queries + embedded migrations
│   │   └── sqlite_identifiers/          # Enums and IDs used in sqlite queries as enums or keys
│   ├── testing/                         # Test data and test utilities
│   └── vendor/                          # Vendored third-party crates (eg. tauri-plugin-http)
├── frontend/                            # Nx typescript monorepo for the Tauri desktop app frontend
│   ├── apps/
│   │   └── artcraft/                    # ArtCraftX the Tauri app's frontend. Used with the `artcraftx` Rust crate.
│   └── libs/                            # Support libraries, reusable React components, etc.
├── script/
│   └── artcraftx/                       # Launch and build scripts for the desktop app
└── Cargo.toml                           # Rust monorepo workspace
```

## Code Style

- Rust with no minimum supported version
- A mix of wreq and reqwest for Rust HTTP clients
- SQLx for SQLite (the desktop app's local task database in `sqlite_database`)
- Never use `println!` or `eprintln!` outside of tests; use `log` crate macros instead
- When two crates export the same type name, alias imports with a suffix: `use foo::Bar as BarFoo;`
- Prefer `use` imports over inline fully-qualified paths; only qualify inline for true one-offs or std collisions
- TypeScript with Nx, React, Vite, Zustand, and Three.js
- Use two spaces for indentation

### File Layout

Organize for top-to-bottom reading. Important things first, details later.

- **Constants** at the top (after imports)
- **Structs/enums** next; outer structs above inner sub-structs
- **API types** in order: Request, Response, Error
- **In impl blocks**: constructors first, then public methods, then private helpers
- Private helpers go *below* the methods that call them
- Among helpers: meatier logic above leaf-level formatters
- **In test modules**: constants first, then test cases (grouped into sub-modules when 2+), then helper functions last

## Markdown

- **Tables must be space-padded so columns align in plain text.** Markdown
  tables are read raw (terminals, diffs, editors) at least as often as
  rendered, and condensed tables are unreadable there. Pad every cell to its
  column width:

  ```markdown
  | Model        | Configuration | Credits    | Speed | Score   |
  |--------------|---------------|--------------------|---------|
  | Meshy 6      | text or image | 104        | 80.0  | +24     |
  | Rodin 2.5    | text or image | 13         | 10.0  | +3      |
  ```

  Not: `| Model | Configuration | Credits | Speed | Score |` packed tight
  with varying widths per row.

---
> Source: [storytold/artcraftx](https://github.com/storytold/artcraftx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
