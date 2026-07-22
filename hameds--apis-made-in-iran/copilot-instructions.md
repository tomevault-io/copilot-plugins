## apis-made-in-iran

> This is a curated list project (not code). The entire repository is a single `README.md` file containing Persian-language documentation of Iranian APIs.

# AGENTS.md

## Overview
This is a curated list project (not code). The entire repository is a single `README.md` file containing Persian-language documentation of Iranian APIs.

## Repository Type
- **No build/test/lint commands** - this is a markdown documentation project
- **Primary file**: `README.md` (276 lines, Persian)

## Workflow

### Link Checking
- GitHub Action runs on every push (`checklinks.yml`)
- Checks all API links in README.md
- **Note**: Iranian sites accessible within Iran may be falsely flagged as broken by the action (due to geo-restrictions). This is expected and not a real issue.

### Adding New APIs
1. Open an Issue or submit a PR
2. Add new entries at the **end of the appropriate category**
3. If uncategorized, add to "نیازمند دسته بندی" section

### Pricing Icons
| Icon | Meaning |
|------|---------|
| :free: | Completely free |
| :moneybag: | Completely paid |
| :free::moneybag: | Both free and paid options |
| :question: | Pricing unknown |

## File Structure
- `README.md` - Main documentation (all content)
- `.github/workflows/checklinks.yml` - Link validation
- `.gitignore` - Standard gitignore
- `LICENSE` - Repository license

---
> Source: [Hameds/APIs-made-in-Iran](https://github.com/Hameds/APIs-made-in-Iran) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
