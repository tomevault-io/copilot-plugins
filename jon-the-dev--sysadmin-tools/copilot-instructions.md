## sysadmin-tools

> This repository contains a collection of system administration and automation tools written in Python.

# CLAUDE.md

## Repository Overview

This repository contains a collection of system administration and automation tools written in Python.

## Project Structure

- **py_dns.py** - DNS MX record checker for individual domains
- **py_dns_from_list.py** - Batch DNS MX record lookup from email list with CSV export
- **gh_org_enable_dependabot.py** - GitHub organization automation to enable Dependabot across repositories

## Technology Stack

- **Python Version**: 3.10
- **Key Dependencies**:
  - `dnspython` - DNS toolkit for Python
  - `pandas` - Data manipulation and CSV export
  - `requests` - HTTP library for GitHub API interactions

## Tool Details

### DNS Tools

1. **py_dns.py**
   - Simple DNS MX record checker
   - Takes domain as command-line argument
   - Prints MX records to console

2. **py_dns_from_list.py**
   - Batch processes email addresses from `emails.txt`
   - Extracts domains from email addresses
   - Performs MX record lookups using Google DNS (8.8.8.8)
   - Exports results to `emails2.txt` as CSV
   - Includes error handling for failed lookups
   - Rate-limited with 200ms delay between requests

### GitHub Automation

**gh_org_enable_dependabot.py**
   - Automatically enables Dependabot across all repositories in a GitHub organization
   - Creates a new branch `enable-dependabot`
   - Adds `dependabot.yml` configuration for GitHub Actions
   - Creates pull request for review
   - Usage: `python gh_org_enable_dependabot.py <org> <token>`

## Development Setup

The project uses Pipenv for dependency management:

```bash
pipenv install
pipenv shell
```

## Usage Examples

```bash
# Check DNS MX records for a domain
python py_dns.py example.com

# Batch process email list
python py_dns_from_list.py

# Enable Dependabot across organization
python gh_org_enable_dependabot.py myorg ghp_token123
```

## Notes for AI Assistants

- This is a utilities repository with standalone scripts
- No unified framework or application structure
- Each script is meant to be run independently
- Credentials and tokens are passed as command-line arguments (not stored)
- The repository has Dependabot enabled for automated dependency updates
- Python version is pinned to 3.10

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-the-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
