## znai

> Znai is a tool to author and generate user guide documentation from markdown files.

# Znai

Znai is a tool to author and generate user guide documentation from markdown files.
It consist of ReactJS (@znai-reactjs) for rendering content and Java for parsing markdown with custom
extensions.

# DocElement

Instead of generating html from md files directly, znai generates DocElement pieces and stores them as JSON. ReactJS App displays 
JSON pieces at runtime. This allows UI to customize rendering depending on the context. E.g. it can render it as part of search preview. Or
render as presentation mode.

# Build Commands

To run tests always use `mvn` command. Do not use `mvnw`.

---
> Source: [testingisdocumenting/znai](https://github.com/testingisdocumenting/znai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
