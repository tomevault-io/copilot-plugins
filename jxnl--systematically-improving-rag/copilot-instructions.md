## coding-style

> - Never use emojis in code, comments, or console output

# Coding Style Guidelines

## General Rules
- Never use emojis in code, comments, or console output
- Write at a 9th-grade reading level
- Always use `uv` instead of `pip` for Python package management
- Use async over synchronous code when possible in Python
- When things can be parallelized, mention it and ask for preferred method

## CLI Applications
- Use `typer` for command-line interfaces instead of `argparse`
- Use `rich` for console output, progress bars, and formatting
- Keep console output simple and readable
- Use consistent text indicators instead of emojis:
  - "Loading..." instead of loading spinner emojis
  - "Success:" instead of checkmark emojis  
  - "Error:" instead of error emojis
  - "Warning:" instead of warning emojis

## Console Output Style
```python
from rich.console import Console
from rich.progress import track
import typer

console = Console()

# Good examples:
console.print("Loading data...", style="blue")
console.print("Success: Data loaded successfully", style="green")
console.print("Error: Failed to connect", style="red")
console.print("Warning: Large dataset detected", style="yellow")

# Use rich for progress tracking
for item in track(items, description="Processing..."):
    process(item)
```

## Project Structure
- Main data processing scripts should be in [latest/capstone_project/utils/](mdc:latest/capstone_project/utils/)
- Use the [WildChatDataLoader](mdc:latest/capstone_project/utils/dataloader.py) for loading conversation data
- Follow the pattern established in [load_to_chromadb.py](mdc:latest/capstone_project/utils/load_to_chromadb.py) for CLI scripts

## Dependencies
- Always use `uv add` for adding new dependencies
- Prefer these libraries:
  - `typer` for CLI applications
  - `rich` for console output
  - `pydantic` for data validation
  - `chromadb` for vector database operations

---
> Source: [jxnl/systematically-improving-rag](https://github.com/jxnl/systematically-improving-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
