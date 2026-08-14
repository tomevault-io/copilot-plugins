## riaak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RIAAK (Richie's Impactful Animal Advocacy Knowledgebase) is an Obsidian vault published as a static website using the Eleventy static site generator, with semantic search powered by Pinecone vector database and OpenAI embeddings. The project uses a dual-deployment strategy:
- Frontend: Deployed to Netlify via GitHub Pages
- Backend: Serverless API deployed to Vercel
- Search Infrastructure: Pinecone vector database with OpenAI embeddings

## Development Commands

### Local Development
```bash
npm install              # Install Node dependencies
npm start               # Start dev server with watch mode (runs Sass + Eleventy in parallel)
```

### Building
```bash
npm run build           # Production build (fetches theme, builds Sass + Eleventy)
npm run prebuild        # Cleans dist directory (runs automatically before build)
```

### Component Commands
```bash
npm run build:eleventy  # Build Eleventy only (production mode)
npm run build:sass      # Build Sass only (compressed)
npm run get-theme       # Fetch remote theme CSS
```

### Python (Search Infrastructure)
```bash
pip install -r requirements.txt   # Install Python dependencies
python scripts/ingest.py          # Process vault and update Pinecone embeddings
```

## Architecture

### Static Site Generation (Eleventy)
- **Input**: `src/site/` - Contains Nunjucks templates, Markdown notes, and assets
- **Output**: `dist/` - Generated static site
- **Notes Directory**: `src/site/notes/` - Contains 400+ Markdown files from Obsidian vault
- **Configuration**: `.eleventy.js` - Extensive Eleventy configuration with custom markdown-it plugins

### Semantic Search Pipeline

**Ingestion (scripts/ingest.py)**:
1. Scans vault and removes vectors for any deleted notes (cleanup step runs first)
2. Walks `src/site/notes/` directory for `.md` files
3. Parses frontmatter using `python-frontmatter`
4. Computes MD5 hash of content to skip unchanged files
5. Chunks notes using `MarkdownHeaderTextSplitter` (splits on H1, H2, H3)
6. Embeds chunks using OpenAI `text-embedding-3-small` (1536 dimensions)
7. Semantic boosting: Injects tags into embedding text as "Context Tags: ..."
8. Upserts vectors to Pinecone with metadata (file_path, tags, url, text, file_hash)
9. Uses URL-encoded file paths as vector IDs to handle special characters
- Each vector stores metadata: `file_path`, `file_hash`, `text`, `tags`, `url`, plus header info (H1/H2/H3)

**Query API (api/search.py)**:
- Vercel serverless function (Python)
- Accepts GET requests with `?q=<query>` parameter
- Embeds query using same OpenAI model
- Queries Pinecone for top 5 matches with cosine similarity
- Returns JSON with score, text, url, file_path
- CORS-enabled for cross-origin requests

### Deployment

**Frontend (Netlify)**:
- Build command: `npm install && npm run build`
- Publish directory: `dist`
- Redirects API calls to Vercel via `netlify.toml`

**Backend (Vercel)**:
- Deploys `api/search.py` as serverless function
- Route: `/api/search` → `api/search.py`
- Requires `PINECONE_API_KEY` and `OPENAI_API_KEY` environment variables

**CI/CD (GitHub Actions)**:
- `.github/workflows/build.yml` - Validates builds on PRs and main branch pushes
- `.github/workflows/deploy.yml` - Runs `ingest.py` on main branch pushes to update search index

### Markdown Processing

The `.eleventy.js` configuration includes extensive markdown-it customizations:
- **Wikilinks**: Converts `[[filename|title]]` to internal links, resolves to `/notes/slugified-path`
- **Tags**: Converts `#tag` to clickable search tags
- **Callouts**: Obsidian-style callouts (`> [!note]`) with collapsible support
- **Math**: MathJax3 support with `$...$` inline math
- **Mermaid**: Diagrams via `mermaid` code blocks
- **PlantUML**: Diagram support
- **Images**: Responsive image optimization with WebP generation
- **Dataview**: Hides Obsidian dataview inline fields `(key::value)`

### Key Files

- `.env` - Site configuration (name, theme, feature flags)
- `vercel.json` - Vercel routing and CORS headers
- `netlify.toml` - Netlify build settings and redirects
- `plugin-info.json` - Tracks files modified from base Digital Garden template
- `src/helpers/` - Custom utilities for link resolution, file trees, etc.

## Environment Variables

**.env (Frontend)**:
- `SITE_NAME_HEADER` - Site title
- `SITE_BASE_URL` - Base URL for canonical links
- `THEME` - URL to remote CSS theme
- `dgEnableSearch`, `dgShowFileTree`, etc. - Feature flags

**Vercel/GitHub Secrets (Backend)**:
- `PINECONE_API_KEY` - Pinecone vector database API key
- `OPENAI_API_KEY` - OpenAI embeddings API key

## Important Notes

- The Pinecone index is named `"digital-garden"` and uses cosine metric
- File paths with special characters are URL-encoded for Pinecone IDs (e.g., `Str%C3%A4ssner.md`)
- The ingestion script uses file hashing to avoid re-embedding unchanged notes (cost optimization)
- Dead links (missing note files) render as unresolved with `/404` href
- Notes tagged with `gardenEntry` in frontmatter become the homepage (`/`)
- Images are optimized to WebP/JPEG at multiple sizes unless `USE_FULL_RESOLUTION_IMAGES=true`

---
> Source: [Thomas-Richardson/RIAAK](https://github.com/Thomas-Richardson/RIAAK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
