## sloppylint

> Sloppy detects "AI slop" patterns in Python code - over-engineering, hallucinations,

# Sloppy - Python AI Slop Detector

## Project Overview

Sloppy detects "AI slop" patterns in Python code - over-engineering, hallucinations, 
dead code, and anti-patterns commonly produced by LLMs. Inspired by KarpeSlop.

**Core concept**: Three axes of AI slop detection:
1. **Noise** (Information Utility) - redundant comments, debug code, placeholders
2. **Lies** (Information Quality) - hallucinated imports, wrong APIs, assumptions
3. **Soul** (Style/Taste) - overconfident comments, god functions, deep nesting

## Project Structure

```
sloppy/
├── src/sloppy/
│   ├── __init__.py          # Package init, version
│   ├── __main__.py          # Entry point: python -m sloppy
│   ├── cli.py               # Argument parsing (argparse)
│   ├── detector.py          # Main orchestration
│   ├── scoring.py           # Slop index calculation
│   ├── reporter.py          # Terminal + JSON output
│   ├── config.py            # pyproject.toml / .sloppyrc support
│   ├── patterns/
│   │   ├── __init__.py      # Pattern registry
│   │   ├── base.py          # BasePattern class
│   │   ├── noise.py         # Axis 1 patterns
│   │   ├── hallucinations.py # Axis 2 patterns
│   │   ├── style.py         # Axis 3 patterns
│   │   └── structure.py     # Structural anti-patterns
│   └── analyzers/
│       ├── __init__.py
│       ├── ast_analyzer.py  # AST-based detection
│       ├── import_validator.py # Package existence checks
│       └── duplicate_finder.py # Cross-file similarity
├── tests/
│   ├── conftest.py          # Pytest fixtures
│   ├── test_patterns/       # Pattern-specific tests
│   ├── test_analyzers/      # Analyzer tests
│   └── fixtures/            # Sample Python files with known issues
├── pyproject.toml
├── README.md
├── AGENTS.md
└── .github/workflows/ci.yml
```

## Development Environment

### Prerequisites
- Python 3.9+
- No external dependencies for core (stdlib only)
- Optional: `rich` for pretty output

### Setup
```bash
# Clone and setup
git clone https://github.com/rsionnach/sloppy.git
cd sloppy

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install in editable mode
pip install -e ".[dev]"
```

### Dependencies
Core tool uses **stdlib only**:
- `ast` - Python AST parsing
- `re` - Regex patterns
- `pathlib` - File handling
- `difflib` - Similarity detection
- `importlib.util` - Import validation
- `json` - Config and output
- `argparse` - CLI

Dev dependencies:
- `pytest` - Testing
- `pytest-cov` - Coverage
- `rich` - Pretty output (optional runtime)

## Build & Test Commands

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src/sloppy --cov-report=term-missing

# Run specific test file
pytest tests/test_patterns/test_hallucinations.py -v

# Type checking (if mypy added)
mypy src/sloppy

# Format check
black --check src/ tests/
isort --check-only src/ tests/

# Run the tool locally
python -m sloppy .
python -m sloppy src/ --severity high
python -m sloppy --output report.json
```

## Code Style & Conventions

### Python Style
- Follow PEP 8
- Use type hints for all function signatures
- Docstrings for public functions (Google style)
- No external dependencies in core - stdlib only
- Keep functions < 50 lines where possible

### Pattern Implementation
All detection patterns inherit from `BasePattern`:

```python
from sloppy.patterns.base import BasePattern, Issue, Severity

class MutableDefaultArg(BasePattern):
    """Detect mutable default arguments."""
    
    id = "mutable_default_arg"
    severity = Severity.CRITICAL
    axis = "quality"  # noise | quality | style | structure
    message = "Mutable default argument - use None instead"
    
    def check_node(self, node: ast.FunctionDef) -> list[Issue]:
        issues = []
        for default in node.args.defaults:
            if isinstance(default, (ast.List, ast.Dict, ast.Set)):
                issues.append(self.create_issue(node, code=ast.dump(default)))
        return issues
```

### AST Visitor Pattern
Use `ast.NodeVisitor` subclasses for traversal:

```python
class SlopVisitor(ast.NodeVisitor):
    def __init__(self, patterns: list[BasePattern]):
        self.patterns = patterns
        self.issues: list[Issue] = []
    
    def visit_FunctionDef(self, node):
        for pattern in self.patterns:
            if hasattr(pattern, 'check_function'):
                self.issues.extend(pattern.check_function(node))
        self.generic_visit(node)
```

### Regex Patterns
Comment/string patterns use compiled regex:

```python
PATTERNS = {
    'overconfident': re.compile(
        r'#\s*(obviously|clearly|simply|just|easy|trivial)\b',
        re.IGNORECASE
    ),
    'hedging': re.compile(
        r'#\s*(should work|hopefully|probably|might|try this)\b',
        re.IGNORECASE
    ),
}
```

## Testing Guidelines

### Test Organization
- One test file per pattern category
- Use fixtures for sample Python code
- Test both detection AND non-detection (no false positives)

### Fixture Format
Create sample files in `tests/fixtures/`:

```python
# tests/fixtures/mutable_defaults.py
def bad_func(items=[]):  # Should detect
    items.append(1)
    return items

def good_func(items=None):  # Should NOT detect
    if items is None:
        items = []
    return items
```

### Test Pattern
```python
def test_mutable_default_detected(fixture_path):
    detector = Detector()
    issues = detector.scan(fixture_path / "mutable_defaults.py")
    
    mutable_issues = [i for i in issues if i.pattern_id == "mutable_default_arg"]
    assert len(mutable_issues) == 1
    assert mutable_issues[0].line == 2

def test_none_default_not_flagged(fixture_path):
    # Ensure good patterns don't trigger false positives
    ...
```

## CLI Interface Specification

```
usage: sloppy [-h] [--output FILE] [--format {compact,detailed,json}]
              [--severity {low,medium,high,critical}]
              [--ignore PATTERN] [--disable PATTERN_ID]
              [--strict | --lenient] [--ci] [--max-score N]
              [--version]
              [paths ...]

Python AI Slop Detector

positional arguments:
  paths                 Files or directories to scan (default: .)

options:
  -h, --help            Show help message
  --output FILE         Write JSON report to FILE
  --format FORMAT       Output format: compact, detailed, json
  --severity LEVEL      Minimum severity to report
  --ignore PATTERN      Glob pattern to exclude (repeatable)
  --disable ID          Disable specific pattern (repeatable)
  --strict              Report all severities
  --lenient             Only critical and high
  --ci                  CI mode: exit 1 if issues found
  --max-score N         Exit 1 if slop score exceeds N
  --version             Show version
```

## Scoring System

```python
SEVERITY_WEIGHTS = {
    Severity.CRITICAL: 30,
    Severity.HIGH: 15,
    Severity.MEDIUM: 8,
    Severity.LOW: 3,
}

def calculate_score(issues: list[Issue]) -> SlopScore:
    return SlopScore(
        noise=sum(w for i in issues if i.axis == "noise"),
        quality=sum(w for i in issues if i.axis == "quality"),
        style=sum(w for i in issues if i.axis == "style"),
        structure=sum(w for i in issues if i.axis == "structure"),
    )
```

## Pattern Registry (46 total)

### Axis 1: Noise (10)
- `redundant_comment`, `empty_docstring`, `generic_docstring`
- `debug_print`, `debug_breakpoint`, `commented_code_block`
- `excessive_logging`, `redundant_pass`, `obvious_type_comment`, `changelog_in_code`

### Axis 2: Quality/Hallucinations (14)
- `hallucinated_import`, `wrong_stdlib_import`, `hallucinated_method`
- `deprecated_api`, `todo_placeholder`, `pass_placeholder`, `ellipsis_placeholder`
- `notimplemented_placeholder`, `assumption_comment`
- `magic_number`, `magic_string`, `mutable_default_arg`
- `wrong_exception_type`, `impossible_condition`

### Axis 3: Style (10)
- `overconfident_comment`, `hedging_comment`, `apologetic_comment`
- `god_function`, `deep_nesting`, `nested_ternary`
- `single_letter_var`, `inconsistent_naming`, `overlong_line`, `multiple_statements`

### Structural (12)
- `single_method_class`, `unnecessary_inheritance`, `god_class`
- `unused_import`, `star_import`, `circular_import_risk`
- `bare_except`, `broad_except`, `empty_except`
- `unreachable_code`, `duplicate_code`, `dead_code`

## Beads Issue Tracking

This project uses [beads](https://github.com/steveyegge/beads) for issue tracking.

```bash
# View all issues
bd list

# View ready work (unblocked)
bd ready

# Create new issue
bd create -t "Title" -p 1 -l "label"

# Complete an issue
bd done <id>
```

## Common Tasks

### Adding a New Pattern
1. Choose category: `noise.py`, `hallucinations.py`, `style.py`, or `structure.py`
2. Create class inheriting `BasePattern`
3. Implement `check_*` method (check_node, check_function, check_line, etc.)
4. Register in `patterns/__init__.py`
5. Add tests in `tests/test_patterns/`
6. Add fixture file if needed

### Adding AST-Based Detection
```python
# In ast_analyzer.py
def visit_ExceptHandler(self, node):
    if node.type is None:  # bare except
        self.report("bare_except", node)
    self.generic_visit(node)
```

### Adding Regex-Based Detection
```python
# In patterns/noise.py
class DebugPrint(BasePattern):
    id = "debug_print"
    pattern = re.compile(r'\bprint\s*\([^)]*\)')
    
    def check_line(self, line: str, lineno: int) -> list[Issue]:
        if self.pattern.search(line):
            return [self.create_issue(lineno, code=line.strip())]
        return []
```

## Security Considerations

- Never execute scanned code - AST parsing only
- Validate all file paths (no directory traversal)
- Handle malformed Python files gracefully
- Don't expose full file paths in public reports

## PR Guidelines

- All PRs must pass tests and have no new issues from sloppy itself
- Add tests for new patterns
- Update AGENTS.md if adding new conventions
- Keep commits atomic and well-described

---
> Source: [rsionnach/sloppylint](https://github.com/rsionnach/sloppylint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
