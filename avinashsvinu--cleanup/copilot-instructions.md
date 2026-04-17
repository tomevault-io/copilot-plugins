## cleanup

> This is a beautiful Python-based system cleanup tool with:

# System Cleanup Master - AI Assistant Rules

## Project Overview
This is a beautiful Python-based system cleanup tool with:
- Interactive terminal UI using Rich library
- Automated duplicate detection and removal
- Cache cleaning and file organization
- macOS-specific optimizations
- Makefile for easy command execution

## Code Style Guidelines

### Python
- Use Python 3.7+ features
- Follow PEP 8 style guide
- Use type hints where appropriate
- Prefer f-strings over .format()
- Use pathlib.Path for file operations
- Rich library for all terminal output

### Formatting
- 4 spaces for indentation
- Max line length: 100 characters
- Double quotes for strings
- Descriptive variable names with snake_case
- Class names in PascalCase

## Project Structure
```
cleanup/
├── cleanup_master.py    # Main script with CleanupMaster class
├── Makefile            # Quick commands (make run, make status, etc.)
├── README.md           # User documentation with Mermaid charts
├── VISUALIZATIONS.md   # Detailed visual workflows
├── CLAUDE.md          # Claude AI instructions
├── .venv/             # Python virtual environment
└── .cursorrules       # This file
```

## Key Components

### CleanupMaster Class
- `scan_system()` - Scans for duplicates, caches, large files
- `clean_duplicates()` - Removes duplicate files
- `clean_caches()` - Clears cache directories
- `organize_downloads()` - Sorts files by category
- `clean_system_wide()` - System-level cleanup (requires sudo)
- `show_summary()` - Display cleanup statistics

### Important Methods
- Always use `self.format_bytes()` for size formatting
- Use `self.get_size()` for calculating directory sizes
- All operations must confirm with user before deleting
- Update `self.stats` and `self.total_freed` after operations

## Design Patterns

### User Interaction
- Always show progress bars for long operations
- Use Rich tables for data display
- Color coding: cyan=info, green=success, yellow=warning, red=error
- Confirm destructive operations with Confirm.ask()
- Show preview before deletion

### Safety First
- Never delete without user confirmation
- Keep oldest file when removing duplicates
- Only clean caches > 10MB by default
- Graceful error handling with try/except
- Show clear error messages

### File Operations
- Use pathlib.Path for all paths
- Handle permissions errors gracefully
- Support for interrupted operations
- Verify file existence before operations
- Calculate sizes before and after

## Makefile Commands
When suggesting commands to users, prefer Makefile:
- `make run` - Interactive mode
- `make auto` - Automatic cleanup
- `make status` - Show disk usage
- `make clean-cache` - Clean caches only
- `make organize` - Sort Downloads
- `make full-clean` - Complete cleanup

## Dependencies
- `rich` - Terminal UI, progress bars, tables
- `click` - CLI framework (for future expansion)
- `psutil` - System utilities and disk info

## Testing Approach
- Test with small sample files first
- Verify calculations are accurate
- Check permissions handling
- Test cancellation/interruption
- Validate summary statistics

## Documentation
- Use emoji for visual appeal (🧹 🔍 📊 ✨)
- Include Mermaid diagrams for workflows
- Provide both Python and Makefile examples
- Show expected output/results
- Include troubleshooting tips

## Common Tasks

### Adding New Cache Directory
1. Add to `scan_system()` cache scanning
2. Update size thresholds if needed
3. Add to `clean_caches()` method
4. Update documentation
5. Test with actual cache dir

### Adding File Category
1. Add extension list to `organize_downloads()`
2. Create category folder
3. Update move logic
4. Document in README
5. Add to visualizations

### New Cleanup Operation
1. Create method in CleanupMaster
2. Add to interactive menu
3. Create Makefile target
4. Update statistics tracking
5. Document and visualize

## Error Handling
- Catch specific exceptions (PermissionError, FileNotFoundError)
- Show user-friendly error messages
- Continue on non-critical errors
- Log errors for debugging
- Provide recovery suggestions

## Performance
- Parallel scanning when possible
- Lazy evaluation for large directories
- Stream large file operations
- Cache size calculations
- Optimize regex patterns

## macOS Specific
- Use `~/Library/Caches` for cache location
- Respect System Integrity Protection
- Handle sandboxed app caches
- Use `sudo` only when necessary
- Test on multiple macOS versions

## UI Patterns
```python
# Progress bar
with Progress(console=self.console) as progress:
    task = progress.add_task("[cyan]Description...", total=items)
    # ... work ...
    progress.advance(task)

# Table
table = Table(title="Title", box=box.ROUNDED)
table.add_column("Name", style="cyan")
table.add_row("value")
self.console.print(table)

# Confirmation
if Confirm.ask("Proceed?", default=True):
    # ... do work ...
```

## Git Workflow
- Clear commit messages
- Feature branches for new features
- Test before committing
- Update documentation with code
- Version releases properly

## Future Enhancements Ideas
- Add scheduling/cron support
- Email/notification on completion
- Web dashboard
- Multi-user support
- Cloud backup integration
- Undo/restore functionality
- Smart duplicate detection (content hash)
- Machine learning for file categorization

## When Helping Users
1. Understand their goal first
2. Suggest Makefile commands when possible
3. Explain what will happen before running
4. Show expected results
5. Provide troubleshooting if issues
6. Update docs if adding features
7. Keep the beautiful UI aesthetic

## Security Considerations
- Never store passwords in code
- Use keychain for sensitive data
- Validate file paths
- Sanitize user input
- Check file permissions
- Audit sudo operations

## Remember
- This tool deletes files - safety first!
- Always confirm before destructive operations
- Show clear, beautiful output
- Make it user-friendly
- Keep code maintainable
- Document everything
- Test thoroughly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avinashsvinu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
