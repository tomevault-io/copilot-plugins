## sternenregen

> This document provides instructions for agents working on this project.

# Agent Instructions

This document provides instructions for agents working on this project.

## PDF Generation

The primary artifact of this project is a PDF file generated from the Typst source files. It is crucial that the PDF is generated correctly and without any warnings.

### Installation

Before you can generate the PDF, you need to have `typst` installed. You can install it using `cargo`:

```bash
cargo install --locked typst-cli
```

### Compilation

To compile the PDF, use the following command from the root of the repository:

Alternatively, you can use the `Makefile`:

```bash
make pdf
```

### Warning Resolution

It is **mandatory** to resolve all warnings during the PDF generation process. Do not submit any changes that introduce new warnings or fail to address existing ones.

## Font Information

The project uses custom fonts located in the `fonts` directory. The correct font names are:

- Bookinsanity Remake
- Dungeon Drop Case
- Mr Eaves SC Remake
- Nodesto Caps
- Scaly Sans Caps
- Caveat
- Scaly Sans Remake
- Solbera Imitation
- Zatanna Misdirection

When referencing fonts in the Typst files, make sure to use these exact names.

---
> Source: [kaylon/sternenregen](https://github.com/kaylon/sternenregen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
