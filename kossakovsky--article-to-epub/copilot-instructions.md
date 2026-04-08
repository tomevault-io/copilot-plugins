## article-to-epub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an interactive CLI tool that converts web articles to EPUB format using Playwright for web scraping and OpenAI for content conversion. The tool features a visible browser mode for user feedback and uses Node.js 24's experimental TypeScript support.

## Development Commands

```bash
# Run the application
npm start

# Install Chromium browser (runs automatically after npm install)
npx playwright install chromium
```

## Architecture

### Pipeline Flow

The application follows a 4-step linear pipeline orchestrated by `src/index.ts`:

1. **User Input** (`src/cli.ts`) - Collects article metadata via interactive prompts
2. **Web Scraping** (`src/scraper.ts`) - Extracts article content using Playwright
3. **Content Conversion** (`src/converter.ts`) - Converts HTML to Markdown via OpenAI API
4. **EPUB Generation** (`src/epub-generator.ts`) - Creates EPUB file from Markdown

### Key Architecture Decisions

**No Build Step**: The project uses Node.js 24's `--experimental-strip-types` flag to run TypeScript directly without transpilation. All imports must use `.js` extensions (not `.ts`) despite the files being TypeScript.

**Visible Browser Mode**: Playwright runs with `headless: false` and `slowMo: 100` to provide visual feedback during scraping. This is intentional UX design, not a development artifact.

**Content Selector Strategy**: The scraper (`src/scraper.ts`) tries multiple semantic HTML selectors in priority order (`article`, `main`, etc.) before falling back to `body`. This handles diverse article layouts.

**Markdown Conversion**: Uses a custom regex-based converter in `src/epub-generator.ts:markdownToHtml()` instead of a library. This is intentionally minimal - if more advanced Markdown features are needed, consider integrating the `marked` library.

**Model Selection**: Uses `gpt-5-mini` model in `src/converter.ts:33` for cost-effective conversion. This is a placeholder - verify the actual model name exists in OpenAI's API or update to `gpt-4o-mini` or similar.

### Logging System

The custom logger (`src/logger.ts`) provides:
- Step-by-step progress tracking
- Spinner animations for long operations
- Colored output (chalk) with semantic meaning (blue=info, green=success, red=error, yellow=warning)

When adding new async operations, follow the pattern:
```typescript
logger.startSpinner('Starting operation');
// ... do work
logger.succeedSpinner('Operation complete');
```

### Error Handling Patterns

All async pipeline functions follow this pattern:
- Wrap operations in try-catch
- Stop spinners with `failSpinner()` before throwing
- Throw descriptive errors with context
- Main function in `index.ts` catches all and exits with code 1

### Environment Configuration

Requires `OPENAI_API_KEY` in `.env` file. The application validates this at runtime in `src/index.ts:37-43` before starting the conversion step.

### File Organization

- `epub/` - Output directory for generated EPUB files (gitignored except `.gitkeep`)
- Generated filenames are sanitized versions of article titles (spaces/special chars → underscores, lowercase)

## Common Development Patterns

### Adding a New Scraping Selector

Edit `src/scraper.ts:59-67` to add new selectors to the priority list. More specific selectors should come first.

### Changing Browser Behavior

Browser launch options are in `src/scraper.ts:24-27`. The `slowMo` parameter controls action delay (milliseconds). The `waitUntil` strategy in `goto()` can be adjusted for different page load requirements.

### Modifying Markdown Conversion Rules

Add new conversion rules in `src/epub-generator.ts:68-91`. Order matters - more specific patterns should be processed before general ones.

## TypeScript Configuration

The project uses ESM modules (`"type": "module"` in package.json) with no tsconfig.json. TypeScript types are checked by the editor but not enforced at runtime. Import statements must use `.js` extensions:

```typescript
// Correct
import { logger } from './logger.js';

// Incorrect
import { logger } from './logger.ts';
import { logger } from './logger';
```

## Dependencies

- **playwright** - Web scraping with Chromium browser automation
- **openai** - GPT API client for content conversion
- **inquirer** - Interactive CLI prompts with validation
- **epub-gen-memory** - In-memory EPUB generation (no temp files)
- **chalk** - Terminal color output
- **ora** - Spinner animations
- **dotenv** - Environment variable loading

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kossakovsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kossakovsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
