## dorkagent

> This document defines code conventions for developers contributing to the DorkAgent project.

# Code Conventions

This document defines code conventions for developers contributing to the DorkAgent project.

## Table of Contents
1. [Python Code Style](#python-code-style)
2. [Naming Conventions](#naming-conventions)
3. [Documentation](#documentation)

## Python Code Style

### Import Order
```python
# 1. Standard library
import sys, re, os, pyfiglet
from datetime import datetime

# 2. Third-party libraries
from dotenv import load_dotenv
from crewai import Crew, LLM, Task, Agent
from langchain_openai import ChatOpenAI
from termcolor import colored
from prompt_toolkit import prompt
from prompt_toolkit.completion import PathCompleter
from crewai_tools import SerperDevTool, ScrapeWebsiteTool

# 3. Local modules
from local_module import function
```

### Code Formatting
```python
# Good example
def verify_api_key(llm_type):
    """Verify API key validity"""
    required_keys = ["SERPER_API_KEY"]

    if llm_type == "openai":
        required_keys.append("OPENAI_API_KEY")

    return required_keys

# Bad example
def verify_api_key(llm_type):
    required_keys=["SERPER_API_KEY"]
    if llm_type=="openai": required_keys.append("OPENAI_API_KEY")
    return required_keys
```

## Naming Conventions

### Functions and Variables
- **Function names**: Use snake_case
  ```python
  def send_notification(message):  # Good
  def sendNotification(message):   # Bad
  ```

- **Variable names**: Use snake_case
  ```python
  target_domain = "example.com"    # Good
  targetDomain = "example.com"     # Bad
  ```

- **Constants**: Use uppercase with underscores
  ```python
  MAX_RETRIES = 3
  DEFAULT_TIMEOUT = 30
  ```

### Classes
- **Class names**: Use PascalCase
  ```python
  class DorkAgent:      # Good
  class dork_agent:     # Bad
  ```

### File Names
- All Python files use lowercase with underscores
- Examples: `dorkagent.py`, `dorkagent_cli.py`

## Documentation

### Docstring Style
- Use single-line docstrings for all functions and classes
- Keep docstrings concise and descriptive

```python
def adjust_depth(target_domains, depth):
    """Adjust domain search depth based on specified depth level"""
    pass

def verify_api_key(llm_type):
    """Verify required API keys are present in environment"""
    pass

class DorkAgent:
    """Main agent class for Google Dorking operations"""
    pass
```

---

---
> Source: [yee-yore/DorkAgent](https://github.com/yee-yore/DorkAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
