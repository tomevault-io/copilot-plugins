## flexvocabtree

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build/Run Commands
- Install dependencies: `pip install -e .`
- Run main script: `python vocabulary_tree.py`
- Test single module: `python -m unittest tests/test_modulename.py`
- Type checking: `mypy vocabulary_tree.py`

## Code Style Guidelines
- **Imports**: Standard library first, then third party (cv2, numpy, etc.), then local imports
- **Formatting**: 4-space indentation, 100-char line length max
- **Types**: Use type hints for function parameters and return values (e.g., `def func(param: int) -> list[str]:`)
- **Naming**: 
  - Functions/variables: snake_case
  - Classes: CamelCase
  - Constants: UPPER_CASE
- **Error handling**: Use explicit try/except blocks with specific exceptions
- **Documentation**: Docstrings for classes and non-trivial functions using numpy style
- **Static class variables**: Prefix with underscore (e.g., `_nodes`)
- **Instance variables**: Prefix with underscore and use properties when needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hcmarchezi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
