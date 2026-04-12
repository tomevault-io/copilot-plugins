## webco

> - Run web crawler: `python3 webcrawler.py`

# Web Crawler Project (webco) Guidelines

## Development Commands
- Run web crawler: `python3 webcrawler.py`
- Run agent tools: `python3 mytool.py` or `python3 namwonfile.py`
- Install dependencies: `pip install requests beautifulsoup4 markdownify`
- Run single test: `python -m unittest tests/test_file.py::TestClass::test_method`
- Lint code: `flake8 .` or `pylint *.py`
- Type checking: `mypy .`

## Code Style Guidelines
- Use Python 3.12+ features
- Follow PEP 8 style guide
- Use type hints consistently (imports from `typing`)
- Class naming: PascalCase (e.g., `VisitWebpageTool2`)
- Function/variable naming: snake_case (e.g., `file_name`)
- Imports organization:
  1. Standard library imports
  2. Third-party imports (like `requests`, `markdownify`)
  3. Local imports

## Error Handling
- Use try/except blocks with specific exception handling
- Provide informative error messages
- For import errors, suggest installation commands

## Third-party Dependencies
- requests
- beautifulsoup4
- markdownify
- smolagents

## Project Components
- webcrawler.py: Main crawler for websites in server.csv
- mytool.py: Custom tool implementations for web tasks
- namwonfile.py: Agent configuration and execution
- server.csv: URL list for crawling (format: URL,Site Name,Category)

## Agent Configuration
- Uses LiteLLM with Claude 3.5 Sonnet model
- Web tools include VisitWebpageTool2 and FileDownloadTool

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/baltop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/baltop)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
