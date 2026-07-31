## claude-conversation-extractor

> This is a standalone tool that extracts Claude Code conversations from the

# Claude Conversation Extractor - Project Context

## Project Overview

This is a standalone tool that extracts Claude Code conversations from the
undocumented JSONL format in `~/.claude/projects/` and converts them to clean
markdown files. This is the FIRST publicly available solution for this problem.

## Key Goals

- **Professional Quality**: This project needs to be polished and professional -
  it's important for the developer's family
- **Easy Installation**: Setting up PyPI publishing so users can
  `pip install claude-conversation-extractor`
- **Wide Adoption**: Make this the go-to solution for Claude Code users

## Repository Structure

```text
claude-conversation-extractor/
├── extract_claude_logs.py    # Main script
├── setup.py                   # PyPI packaging configuration
├── README.md                  # Professional documentation
├── LICENSE                    # MIT License with disclaimer
├── CONTRIBUTING.md            # Contribution guidelines
├── requirements.txt           # No dependencies (stdlib only)
├── .gitignore                # Python gitignore
└── CLAUDE.md                 # This file
```

## Development Workflow

1. Always create feature branches for new work
2. Ensure code passes flake8 linting (max-line-length=100)
3. Test manually before committing
4. Update version numbers in setup.py for releases
5. Create detailed commit messages

## Current Status

- ✅ Core functionality complete and tested
- ✅ Professional documentation
- ✅ Published to GitHub:
  <https://github.com/ZeroSumQuant/claude-conversation-extractor>
- 🚧 Setting up PyPI publishing
- 📋 TODO: Add tests, CI/CD, screenshots

## PyPI Publishing Setup (In Progress)

1. Update setup.py with proper metadata
2. Create pyproject.toml for modern packaging
3. Set up GitHub Actions for automated publishing
4. Register on PyPI and get API token
5. Configure repository secrets

## Testing Commands

```bash
# Test extraction
python3 extract_claude_logs.py --list
python3 extract_claude_logs.py --extract 1

# Lint check
python3 -m flake8 extract_claude_logs.py --max-line-length=100

# Test installation
pip install -e .
```

## Important Notes

- No external dependencies (uses only Python stdlib)
- Supports Python 3.8+
- Cross-platform (Windows, macOS, Linux)
- Read-only access to Claude's conversation files
- Includes legal disclaimer for safety

## Marketing/Sharing Plan

- Anthropic Discord
- r/ClaudeAI subreddit
- Hacker News
- Twitter/X with relevant hashtags
- Create demo GIF showing the tool in action

## Version History

- 1.0.0 - Initial release (planned)
  - Core extraction functionality
  - Multiple output formats
  - Batch operations

---
> Source: [ZeroSumQuant/claude-conversation-extractor](https://github.com/ZeroSumQuant/claude-conversation-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
