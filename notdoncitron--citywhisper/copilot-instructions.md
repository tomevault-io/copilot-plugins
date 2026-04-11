## citywhisper

> The `jcodemunch-mcp` server is installed and configured globally. Use it for all symbol-level code lookups to save tokens and improve context accuracy.

# Gemini: CityWhisper

## 🤖 Codebase Exploration (jCodeMunch)
The `jcodemunch-mcp` server is installed and configured globally. Use it for all symbol-level code lookups to save tokens and improve context accuracy.

### Workflow:
1. **List Repos:** Call `list_repos` to see if the project is already indexed.
2. **Index Folder:** If not indexed, call `index_folder` with the current directory path (`C:\Users\pasca\busineesspowerpoint`).
3. **Search Symbols:** Use `search_symbols` to find classes, functions, or variables.
4. **Get Symbol:** Use `get_symbol` to retrieve the source code for a specific symbol.
5. **Fallbacks:** Only use `read_file` when editing or if MCP is unavailable.

## 💰 0-Cost Stack (Current)
- **Logic:** Groq (Llama-3.3-70b-versatile)
- **Audio:** edge-tts (Neural Voices)
- **Maps:** Leaflet + OpenStreetMap
- **Data:** Wikipedia API, Rijksmuseum API, PDOK API

## 🧪 Testing
- **Backend:** Run `pytest` from the `backend/` directory using the `venv`.
- **UI:** Use `chrome-devtools` for visual validation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NotDonCitron) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
