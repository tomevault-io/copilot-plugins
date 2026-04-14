## scrapero

> Check git status then if any changes, add Commit and push all changes.


```

# User Commands (prefix with --):

--GCP  
Check git status then if any changes, add Commit and push all changes.

--MIND  
Before any action, remind yourself:

- This isn't fucking enterprise. We're indiehackers, building MVPs—move fast, stay practical.
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- YAGNI (You Aren't Gonna Need It)
- SOLID (Single responsibility, Open-closed, Liskov, Interface segregation, Dependency inversion)
- FCP (Favor Composition over Inheritance)
- PoLA (Principle of Least Astonishment)
- SoT (Single Source of Truth)

--WD  
Run --X and --MIND and --GCP.
Do a quick, MVP-level review of the codebase. regarding what's described along with the command.
Write an actionable checklist in /dev-docs/REVIEW-\*\*\*.md.  
Don't touch code or other docs.  
When done, --GCP then --RING

--AP  
Run --X and --MIND and --GCP  
Take the latest review and break it down into a simple, step-by-step action plan with checkboxes—keep splitting steps until atomic.  
Save as /dev-docs/ACTION-PLAN-\*\*\*.md, then --GCP  then --RING

--EXE  
Run --MIND and --GCP then execute the action plan (from file), checking off steps as you go.  
Commit and push (--GCP) after each step.

--TERMINATOR  
Run --EXE, then --DS. then --RING

--CD  
Run --MIND + --GCP  
Find and delete obsolete .md files, then --GCP  then --RING

--DS  
Don't stop until the process is totally finished.

--PUB
--MIND, --GPC then run the tests, correct any error then --GPC and npm publish  then --RING

--X  
Don't touch the codebase, except for the one file specified (if any)  then --RING

--READY? --MIND --X --GCP then --WD about is this codebase 100% production ready, and what's still lacking if not ?  then --RING

--RING run  printf "\a" 


- Is error handling robust and consistent?
- Is test coverage meaningful and adequate?
- Do the tests catch real issues, or are they just for show?
- Are tests run pre-commit?
- Is the README clear, concise, and covers install/run/debug basics?
- Do we have legacy code, dead code, forgotten todos rotting somewhere ?
- Do we have deprecation warnings or other warnings somewhere ?
- Are all the dependencies up to date ?

--OBSO
--MIND, --GPC then search the codebase for duplicated and confusing documentations, non single sources of truth, and --WD about your findings and how we could fix those and making this doc a safe heaven. Then --GPC and point us to the created doc

--DUP
--MIND, --GPC then search the codebase for duplicated and confusing code, that could perhaps be easily consolidated, simplified or grouped. --WD about your findings and how we could fix those and making this doc a safe heaven. Then --GPC and point us to the created doc



# Scraper Development Field Guide

## Using the BaseScraper Class

All scrapers should extend the existing `BaseScraper` class located at:
```
src/scrapers/base/BaseScraper.ts
```

This provides common functionality like:
- Configuration loading
- Pipeline setup
- Crawler initialization
- Logging

## Output Format & Naming Convention

**Final outputs MUST be in CSV format.** You can use JSON, XML, or any format internally for processing, but what goes into the `outputs/` folder must be CSV.

All scraper outputs go to a global `outputs/` folder with this naming format:
```
{datetime}-{state}-{scrapername}.csv
```

Examples:
- `2025-07-08-141530-raw-wix-app-market.csv`
- `2025-07-08-141530-processed-wordpress-plugins.csv`
- `2025-07-08-141530-final-wix-roadmap.csv`

Where:
- **datetime**: YYYY-MM-DD-HHMMSS format
- **state**: `raw`, `processed`, `final`, `partial`, etc.
- **scrapername**: Your scraper's name (lowercase, hyphens)

## Flexible Guidelines

### Output Location
- All outputs go to the global `outputs/` folder
- Use the datetime-state-scrapername naming convention
- Keep scraper directories clean - no data files inside

### Temporary Files
- Use a `.temp/` directory for work-in-progress files
- Can store JSON, XML, or any format here for processing
- Clean these up when done

### Configuration
- Each scraper needs a `config.json`
- Document what your scraper does in a README.md

### States You Can Use
- `raw` - Direct from the source, unprocessed
- `processed` - Cleaned up, deduplicated
- `partial` - Incomplete scrape (stopped early)
- `final` - Ready for use
- `test` - Test runs with limited data

## Simple Checklist

When creating a new scraper:

- [ ] Extend BaseScraper class
- [ ] Output to `outputs/` folder 
- [ ] Use datetime-state-scrapername format
- [ ] Create a README.md
- [ ] Add config.json
- [ ] Clean up temp files when done

## Field Documentation

Document your output fields in README.md:

```markdown
## Output Fields

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| id | string | Unique identifier | "app-123" |
| name | string | Display name | "My App" |
| url | string | Product URL | "https://..." |
| scraped_at | ISO 8601 | When scraped | "2025-07-08T14:15:30Z" |
```

## Example Usage

```javascript
// Extending BaseScraper (TypeScript)
import { BaseScraper } from '../base/BaseScraper';

class MyMarketplaceScraper extends BaseScraper {
  // Your implementation
}

// Generate output filename
const datetime = new Date().toISOString()
  .replace(/[:.]/g, '-')
  .replace('T', '-')
  .slice(0, -5);
const state = 'raw';
const scraperName = 'my-marketplace';
const outputFile = `outputs/${datetime}-${state}-${scraperName}.csv`;
```

## Key Points

1. **Extend BaseScraper** - Don't reinvent the wheel
2. **Use the naming convention** - `datetime-state-scrapername`
3. **Keep it simple** - We're building MVPs, not enterprise software
4. **Document your fields** - Help others understand your data

Remember: Move fast, stay practical. This isn't enterprise - we're indiehackers building MVPs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davstr1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davstr1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
