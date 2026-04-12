## sotd-pipeline

> Always organize imports in this order:

# Python-Specific Rules for SOTD Pipeline

## Import Organization
Always organize imports in this order:
1. Standard library imports
2. Third-party imports  
3. Local application imports

```python
import re
from pathlib import Path
from typing import Optional

import yaml
import praw

from sotd.match.base_matcher import BaseMatcher
```

## Type Hints Best Practices
- Use modern type hints (Python 3.9+): `list[dict]` instead of `List[Dict]`
- Use Optional for nullable values
- Use Path for file system operations
- Always include return type annotations

```python
def process_comments(comments: list[dict]) -> dict[str, list[str]]:
    """Process SOTD comments and extract product mentions."""
    pass

def find_product(text: str) -> Optional[str]:
    """Find product mention in text."""
    pass

def load_catalog(catalog_path: Path) -> dict[str, dict]:
    """Load product catalog from YAML file."""
    pass
```

## CLI Argument Patterns
Standard CLI argument patterns for phases:

```python
import argparse

def setup_cli_args():
    parser = argparse.ArgumentParser(description="SOTD Pipeline Phase")
    parser.add_argument('--month', required=True, 
                       help='Month to process (YYYY-MM format)')
    parser.add_argument('--verbose', action='store_true',
                       help='Enable verbose logging')
    parser.add_argument('--dry-run', action='store_true',
                       help='Show what would be done without executing')
    return parser
```

## Logging Rules

### **MANDATORY**: Use Logging Exclusively

**Always use logging instead of print statements for all output that should appear in logs.**

- **Use `logger.info()`, `logger.debug()`, `logger.warning()`, `logger.error()`** for all log messages
- **Never use `print()` statements** for log output (except for temporary debugging that will be removed)
- **Print statements bypass the logging formatter** and won't have timestamps, level indicators, or proper formatting
- **In Docker mode, stdout/stderr are redirected to log files**, so print statements will appear without proper formatting

**Exceptions:**
- Terminal-only output (like pipeline summaries, progress bars) can use print
- Temporary debugging print statements that will be removed before commit

**Logging Setup:**
```python
import logging
from sotd.utils.logging_config import setup_pipeline_logging

# In main() function:
def main(argv: Sequence[str] | None = None) -> int:
    # Setup logging - reads LOG_LEVEL env var (default: INFO)
    setup_pipeline_logging(level=logging.INFO)
    
    try:
        parser = get_parser()
        args = parser.parse_args(argv)
        
        # --debug flag overrides LOG_LEVEL env var (CLI takes precedence)
        if args.debug:
            logging.getLogger().setLevel(logging.DEBUG)
        
        # Use logger, not print
        logger = logging.getLogger(__name__)
        logger.info("Processing started")
        logger.debug("Debug information")
        logger.error("Error occurred")
        
        # ❌ WRONG - Don't do this:
        # print("[INFO] Processing started")
        # print(f"[ERROR] Error: {e}")
        
        # ✅ CORRECT - Do this:
        # logger.info("Processing started")
        # logger.error(f"Error: {e}")
```

**Log Format:**
All log messages automatically use standardized format: `[YYYY-MM-DD HH:MM:SS] [LEVEL] MESSAGE`
- Don't include level indicators in message text (formatter adds them)
- Don't include timestamps in message text (formatter adds them)

## File I/O Rules
- Use UTF-8 encoding explicitly for all file operations
- Use yaml.safe_load() for YAML files
- Pretty-print JSON output for human readability
- Use atomic writes for data files to prevent corruption

```python
def save_json_data(data: dict, file_path: Path):
    """Save JSON data with proper formatting."""
    # Create temporary file for atomic write
    temp_path = file_path.with_suffix('.tmp')
    with temp_path.open('w', encoding='utf-8') as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
    # Atomic move
    temp_path.replace(file_path)

def load_yaml_config(config_path: Path) -> dict:
    """Load YAML configuration safely."""
    with config_path.open('r', encoding='utf-8') as f:
        return yaml.safe_load(f)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nmrs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nmrs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
