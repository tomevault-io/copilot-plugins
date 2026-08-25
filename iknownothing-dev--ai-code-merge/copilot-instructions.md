## ai-code-merge

> **AI Code Merge** is a dual-interface tool that concatenates source code files into a single markdown document for AI analysis. It processes entire codebases by walking directory trees, filtering files based on patterns/exclusions, and aggregating content into structured markdown output.

# AI Code Merge - Developer Guide for AI Agents

## Project Overview

**AI Code Merge** is a dual-interface tool that concatenates source code files into a single markdown document for AI analysis. It processes entire codebases by walking directory trees, filtering files based on patterns/exclusions, and aggregating content into structured markdown output.

**Key Use Case**: Prepare codebases for input to LLMs by creating comprehensive source snapshots without build artifacts, dependencies, or IDE metadata.

## Architecture

### Package Structure (`src/aicodemerge/`)
- **`cli.py`**: CLI interface supporting local directories and Git URLs
- **`gui.py`**: PyQt5 GUI wrapper with drag-and-drop and Git URL support
- **`git_support.py`**: Isolated Git operations module
  - URL detection, validation, repository cloning
  - Git metadata capture (branch, commit, tags, status)
  - Binary file and LFS pointer detection
  - Accurate .gitignore handling with git check-ignore
- **`config.py`**: Shared configuration constants

### Two-Stage Analysis Pipeline

**Stage 1: Source Resolution** (`resolve_source()`)
- Detects if input is local path or Git URL (HTTPS, SSH, git://)
- For local: validates and returns directory
- For Git: clones to temp directory, captures metadata, returns working dir
- Returns `CloneContext` for proper cleanup management

**Stage 2: Analysis** (`run_analysis()`)
- Isolated pipeline accepting any directory
- 1. Parse configuration (depth, size, patterns)
- 2. Build directory structure tree
- 3. Collect files matching filters
- 4. Process files (detect binary, LFS, validate size)
- 5. Serialize output with Git metadata (if available)

### Core Processing Pipeline

1. **Path Traversal**: Recursive `os.walk()` respects depth limits via `current_depth` counter
2. **Intelligent Filtering**: Two-stage filtering using:
   - `.gitignore`-based patterns (dynamically parsed if exists)
   - Hardcoded `DEFAULT_EXCLUDE_PATTERNS` (310+ patterns covering Node, Python, Java, .NET, iOS, etc.)
   - Include-pattern matching (fnmatch) for `*.py`, `*.js`, etc.
   - Optional: git check-ignore for nested .gitignore accuracy
3. **Binary & LFS Detection** (NEW):
   - Binary: Null byte sampling in first 8KB
   - LFS: Header check for `version https://git-lfs.github.com/spec/v1`
4. **File Validation**: Size checks before appending (skips files >max_size_kb)
5. **Markdown Serialization**: Wraps file content in markdown code blocks with language detection

## New CLI Flags (Git Support)

```
Git Source & Clone Options:
  --branch BRANCH              Git branch to checkout (default: default branch)
  --commit HASH                Git commit hash to checkout
  --clone-depth N              Shallow clone depth (default: 1 for speed)
  --clone-timeout SEC          Clone operation timeout (default: 60 seconds)
  --submodules ignore|init     Submodule handling (default: ignore)
  --auth-token TOKEN           Auth token for private HTTPS repos (not recommended)
  --no-cleanup                 Keep cloned repo for inspection (temp dir location printed)

Advanced Filtering:
  --use-git-check-ignore       Use 'git check-ignore' for nested .gitignore accuracy
  --skip-binary                Skip files with null bytes (binary detection)
  --handle-lfs skip|pointer    LFS file handling (skip or show pointer)
```

All flags fully backward compatible with v1.0. Local analysis unaffected.

## Critical Implementation Details

### Shared Code Distribution
- **Single Source of Truth**: `src/aicodemerge/config.py` defines `DEFAULT_EXCLUDE_PATTERNS`, `DEFAULT_MAX_DEPTH`, `DEFAULT_MAX_SIZE`
- **Imported in Both**: CLI (`cli.py`) and GUI (`gui.py`) import from `aicodemerge.config`
- **Git Module**: `git_support.py` imported only when Git operations needed
- **Backward Compatible**: All v1.0 functionality unchanged, new features optional

### File Size Encoding
Files are read with `encoding="utf-8", errors="ignore"` to handle binary/non-UTF8 content gracefully (no crashes on encoding errors).

### Directory Listing Format
Tree structure uses ASCII art: `│   ` for indentation, `prefix + "│   "` for nested levels—purely cosmetic but hardcoded in two places.

### Git Metadata Capture (NEW)
Captured using subprocess `git` commands:
- Current branch: `git rev-parse --abbrev-ref HEAD`
- Commit hash: `git rev-parse HEAD`
- Short hash: First 7 characters
- Latest tag: `git describe --tags --abbrev=0`
- Dirty status: `git status --porcelain`
- All serialized in markdown header for reproducibility

### Binary & LFS Detection (NEW)
- **Binary Detection**: Sample first 8KB for null bytes
- **LFS Pointer Detection**: Check for `version https://git-lfs.github.com/spec/v1` header
- **Exclusion**: Both types excluded from content by default (configurable)

### Configuration Defaults
- **Max depth**: 4 (limits traversal; prevents deep node_modules exploration)
- **Max file size**: 100 KB (prevents huge outputs)
- **File patterns**: `*` (matches everything after exclusion filter)
- **Clone depth**: 1 (shallow, saves 50-90% bandwidth)
- **Clone timeout**: 60 seconds

## Development Workflows

### Running the CLI (Local Path)
```bash
python -m aicodemerge /path/to/project -d 3 -s 50 -p "*.py,*.js"
```

### Running the CLI (Git URL)
```bash
# Shallow clone, analyze, auto-cleanup
python -m aicodemerge https://github.com/owner/repo.git --with-stats

# Specific branch
python -m aicodemerge https://github.com/owner/repo.git --branch develop

# SSH (requires SSH agent configured)
python -m aicodemerge git@github.com:owner/repo.git
```

### Running the GUI
```bash
python -m aicodemerge.gui
```
Accepts local paths and Git URLs; outputs markdown in specified directory.

### Custom Configuration Mode (CLI only)
```bash
python -m aicodemerge /path -c
```
Launches interactive prompt to add/remove exclusion patterns before processing.

### Testing Edge Cases
- Permission-denied folders: Handled gracefully (`[Permission Denied]` in output)
- Large files: Logged and excluded with size info, not truncated
- Non-UTF8 files: Silently converted via `errors="ignore"`
- Binary files: Detected and skipped with reason
- LFS pointers: Detected and excluded/shown as pointers (configurable)
- Empty directories: Skipped (not shown in tree output)
- Network timeouts: Configurable timeout with error message
- Invalid URLs: Validated and rejected with clear error

## Key Patterns to Preserve

### Pattern: Source Resolution (NEW)
```python
# Always resolve source first, then analyze
context = resolve_source(source, branch=..., timeout=...)
try:
    working_dir = context.working_dir
    metadata = context.metadata
    # ... run_analysis(...) ...
finally:
    cleanup_clone(context)
```
Ensures cleanup happens even on error via `finally` block.

### Pattern: Analysis Isolation (NEW)
```python
# Core analysis takes any directory (local or cloned)
output_str, files_data, config = run_analysis(
    working_dir=working_dir,
    max_depth=...,
    git_metadata=metadata,  # optional
    ...
)
```
Reusable by CLI and GUI, doesn't know about source type.

### Pattern: Conditional Path Filtering
```python
if should_include_file(file_path, gitignore_matcher, patterns):
    # process file
```
Always validate against gitignore rules AND pattern list before processing.

### Pattern: Generator-Based Tree Listing
`list_directory()` returns list of strings (not file objects) to allow preview before processing large trees.

### Pattern: Verbose Logging
Use Python's `logging` module for debug output—configured via `basicConfig()` in main:
```python
logging.basicConfig(
    level=logging.DEBUG if args.verbose else logging.INFO,
    format='%(levelname)s: %(message)s'
)
logger = logging.getLogger(__name__)
logger.debug("detailed message")
```
Never use print statements for diagnostic output; logging allows log-level control and file redirection.

### Pattern: Configuration Separation
Keep default patterns separate from user-provided patterns until merge point (`parse_gitignore()`).

## Notable Design Decisions

### Why Shallow Cloning by Default?
Reduces bandwidth by 50-90% while getting current state; users can set `--clone-depth 0` for full history if needed.

### Why Subprocess Over GitPython?
Minimal dependencies, robust for subprocess isolation, familiar `git` command semantics.

### Why Separate git_support.py?
Isolates Git complexity, reusable by CLI and GUI, easier to test and maintain.

### Why .gitignore + Defaults?
User's .gitignore takes precedence over defaults; fallback to comprehensive hardcoded patterns for robustness.

### Why Exclude Documentation by Default?
`*.md`, `*.rst`, `*.txt` are excluded because:
- LLMs already understand markdown
- README files distract from source code analysis
- Saves space in output

### Why `fnmatch` Over Regex?
Simpler shell-style patterns match `.gitignore` conventions; users expect `*.txt` not `.*\.txt`.

### Why Size Limits?
Prevents memory exhaustion and token overflow in downstream LLM processing; 100 KB default assumes analysis tool has 2-4 KB context for metadata.

## Common Modifications

### Adding a New Exclude Pattern
1. Add to `DEFAULT_EXCLUDE_PATTERNS` in `config.py` (single source of truth)
2. Both CLI and GUI automatically use updated patterns via import
3. Test with: `python -m aicodemerge . -v` to verify pattern filtering
4. Consider `.gitignore` user overrides (they take precedence)

### Extending File Processing
Modify `append_file_content()` to:
- Change markdown syntax (e.g., use ````diff` for diffs)
- Pre-process content (strip comments, normalize whitespace)
- Add metadata headers

### Changing Default Parameters
Edit `parse_arguments()` defaults (CLI) and `initUI()` spinbox/text field defaults (GUI) in parallel.

## Recent Enhancements (v2.0)

### Feature #4: Incremental Output / Filtering by Last Modified
**Status**: ✅ Implemented and tested (5 unit tests + CLI integration test)

Adds `--since <YYYY-MM-DD>` flag to include only files modified after specified date:
- Reduces token usage for LLMs by excluding unchanged files in monorepos
- Implementation: `is_within_date_filter()` checks `os.path.getmtime()` against parsed date
- Filters applied to both directory structure and file contents
- Error handling for invalid date formats with helpful messages

Usage:
```bash
python -m aicodemerge /path/to/project --since 2026-01-26
```

### Feature #5: Project Summary Detection
**Status**: ✅ Implemented and tested (5 unit tests + CLI integration test)

Adds `--include-docs` flag to optionally include documentation files:
- Disabled by default (honors original behavior of excluding markdown)
- `DEFAULT_INCLUDE_DOCS_PATTERNS` in `config.py`: `['README*', 'ARCHITECTURE*']`
- When enabled, README and ARCHITECTURE files prepend to output for LLM context
- Works in combination with other filters (date, size, pattern)

Usage:
```bash
python -m aicodemerge /path/to/project --include-docs
```

### Feature #6: GUI Output Directory Selection
**Status**: ✅ Implemented and tested (manual GUI verification + CLI integration)

GUI now includes:
- `output_dir_input` QLineEdit widget showing selected output directory
- "Browse..." QPushButton to open `QFileDialog.getExistingDirectory()`
- `select_output_directory()` method to handle directory selection
- Default behavior: uses current working directory if empty
- Displays full path for user confirmation

Usage (GUI):
1. Click "Browse..." button next to "Output Directory"
2. Select desired output location in dialog
3. Generate markdown to selected directory

Usage (CLI):
```bash
python -m aicodemerge /path/to/project -o /tmp/output.md
```

## Dependencies
- **Python**: 3.6+
- **PyQt5**: 5.15.4 (GUI only; CLI has zero external dependencies)
- **config.py**: Provides `DEFAULT_EXCLUDE_PATTERNS`, `DEFAULT_INCLUDE_DOCS_PATTERNS`, `DEFAULT_MAX_DEPTH`, `DEFAULT_MAX_SIZE` shared across CLI and GUI

## Implementation Notes (Updated)

### Gitignore Caching (Performance Optimization)
The `parse_gitignore()` function returns a compiled matcher that is cached and reused throughout file traversal, avoiding O(n×m) complexity (patterns × files). This optimization provides 10-50× speedup on large repos.

### Logging Integration
Both CLI and GUI use Python's `logging` module:
- **CLI**: `basicConfig()` called in `main()` with level based on `--verbose` flag
- **GUI**: `basicConfig()` called in `run_aicodemerge()` at INFO level (no verbose equivalent yet)
- Replaces old `log_verbose()` function entirely

## Testing Checklist
- [x] CLI processes a real project directory correctly
- [x] CLI `-v` flag produces DEBUG-level logs; without flag shows INFO only
- [x] Custom config mode (`-c` flag) still works with user pattern editing
- [x] GUI drag-and-drop registers folder path
- [x] Size limits prevent large files from output
- [x] Patterns exclude `.git/`, `node_modules/`, build artifacts
- [x] `.gitignore` file overrides default patterns if present
- [x] Output markdown renders in GitHub/markdown viewers
- [x] `config.py` imports successfully in both CLI and GUI
- [x] **NEW**: `--since <YYYY-MM-DD>` filters files by modification date
- [x] **NEW**: `--include-docs` includes README and ARCHITECTURE files
- [x] **NEW**: GUI output directory selection with browse button
- [x] **NEW**: Combined filters work together (date + docs + size + pattern)
- [x] **NEW**: Git URL support (HTTPS, SSH, git://)
- [x] **NEW**: Git metadata capture (branch, commit, tags)
- [x] **NEW**: Binary file detection
- [x] **NEW**: Git LFS pointer detection
- [x] **NEW**: Shallow cloning with configurable timeout
- [x] **NEW**: Submodule handling (ignore/init)

### Test Coverage (v2.0 - Updated)
Run comprehensive tests:
```bash
# Unit tests for Git support (24 tests)
python3 -m unittest tests.test_git_support -v

# Integration tests (11 tests)
python3 -m unittest tests.test_integration -v

# Run all tests
python3 -m unittest discover -s tests -v
```

**Result**: 100% passing (35 tests + legacy features)
- 24 unit tests for git_support module
- 11 integration tests for complete workflows
- All new Git features tested and verified

---
> Source: [iknownothing-dev/ai-code-merge](https://github.com/iknownothing-dev/ai-code-merge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
