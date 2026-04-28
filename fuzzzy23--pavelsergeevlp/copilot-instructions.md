## pavelsergeevlp

> Обеспечивай доступ к файловой системе для всех типов файлов, используя MCP-сервер "filesystem". Используй:

Обеспечивай доступ к файловой системе для всех типов файлов, используя MCP-сервер "filesystem". Используй:

"tools": [
    "read_file",
    "read_multiple_files",
    "write_file",
    "edit_file",
    "create_directory",
    "list_directory", 
    "directory_tree",
    "move_file",
    "search_files",
    "get_file_info",
    "list_allowed_directories"
]


**ВАЖНО:** При использовании любых инструментов MCP filesystem (`read_file`, `write_file`, `list_directory` и т.д.) всегда указывай **полный абсолютный путь** к файлу или директории, начиная с `C:/Users/ilyam/CursorProject/TgParserGit/`. Относительные пути могут работать некорректно.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fuzzZy23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
