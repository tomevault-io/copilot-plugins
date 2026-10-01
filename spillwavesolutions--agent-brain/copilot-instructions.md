## agent-brain

> Use Agent Brain search before you read files by hand in this project.


This project indexes its docs and code with Agent Brain.

1. Search first. Run `agent-brain query "<question>"` (or the `search_documents` MCP tool) before you open files by hand. Read only the files that the results name.
2. Pick the mode by question type. Use `bm25` for exact symbol names, `vector` for concepts, `hybrid` (the default) when unsure, `graph` for callers and relationships, and `multi` for a broad first pass.
3. Check the server before you search. `agent-brain status` must report HEALTHY. If it does not, run `agent-brain start` from the project root.
4. Keep the index current. After you add or move many files, run `agent-brain index <folder> --include-code`.
5. Never edit files under `.agent-brain/`. That directory is server state.

---
> Source: [SpillwaveSolutions/agent-brain](https://github.com/SpillwaveSolutions/agent-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
