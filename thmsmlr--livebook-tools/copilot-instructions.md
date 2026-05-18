## livebook

> Helpful tips while editing a Livebook .livemd file


Any time you edit a .livemd file.
Call the fetch_livemd_outputs tool to verify that your changes successfully executed.
When you are writing code in an elixir markdown cell, you don't need to `IO.inspect()` or print anything as the last expression will automatically be printed out.
Imports and aliases create in one cell are available in subsequent cells.

---
> Source: [thmsmlr/livebook_tools](https://github.com/thmsmlr/livebook_tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
