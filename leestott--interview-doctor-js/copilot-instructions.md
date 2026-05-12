## interview-doctor-js

> Interview Doctor is an offline, AI-powered interview preparation assistant built with JavaScript, [Foundry Local](https://foundrylocal.ai/), and SQLite. It uses Retrieval-Augmented Generation (RAG) to generate tailored interview questions based on the user's CV/resume and job description.


# Interview Doctor – Agent Instructions

## Project Overview

Interview Doctor is an offline, AI-powered interview preparation assistant built with JavaScript, [Foundry Local](https://foundrylocal.ai/), and SQLite. It uses Retrieval-Augmented Generation (RAG) to generate tailored interview questions based on the user's CV/resume and job description.

**Key constraint: This application runs 100% offline.** Never introduce cloud dependencies, external API calls, or features that require internet connectivity.

## Technology Stack

- **Runtime**: Node.js >= 20 (ES modules)
- **AI**: Foundry Local via `foundry-local-sdk` – native `ChatClient` for inference (no OpenAI shim)
- **Database**: SQLite via `sql.js` – pure JavaScript, no native compilation (TF-IDF vectors for RAG retrieval)
- **PDF**: `pdf-parse` for offline text extraction
- **Web**: Express.js server + single-file HTML frontend (no build step, no framework)
- **Tests**: Node.js built-in test runner (`node:test` + `node:assert/strict`)

## Architecture

```
src/
├── config.js       → Central configuration (model, paths, chunk sizes)
├── chunker.js      → Text chunking + TF-IDF + cosine similarity
├── vectorStore.js  → SQLite-backed vector store for RAG
├── pdfParser.js    → PDF text extraction
├── chatEngine.js   → RAG orchestration + Foundry Local LLM integration
├── prompts.js      → System prompts (full + compact variants)
├── server.js       → Express web server + REST/SSE API
└── ingest.js       → Document ingestion script
```

## Coding Conventions

- **ES modules** throughout (`import`/`export`, `"type": "module"` in package.json)
- **No TypeScript** — plain JavaScript for simplicity and zero build step
- **Single-file frontend** — `public/index.html` contains all HTML, CSS, and JS inline
- **No frameworks** on the frontend — vanilla JavaScript, no React/Vue/etc.
- **Parameterized queries** for all SQLite operations (use `?` placeholders with arrays, never string concatenation)
- **Async VectorStore** — use `await VectorStore.create(dbPath)` factory method (sql.js is async for init)
- **Path sanitization** — all file operations validate paths stay within expected directories
- **Error handling** — Express routes catch and log errors, return appropriate HTTP status codes

## Key Patterns

### Foundry Local Integration
```javascript
import { FoundryLocalManager } from "foundry-local-sdk";

const manager = FoundryLocalManager.create({ appName: "interview-doctor" });
const model = await manager.catalog.getModel("phi-3.5-mini");

// Download if not already cached
if (!model.isCached) {
  await model.download((progress) => console.log(`${Math.round(progress)}%`));
}

await model.load();
const chatClient = model.createChatClient();
```

### RAG Flow
1. User query → `termFrequency()` → TF-IDF vector
2. Cosine similarity against all chunks in SQLite → top-K results
3. Build prompt: system prompt + retrieved context + user question
4. Send to Foundry Local via native `ChatClient`
5. Stream response back to user

### Document Ingestion
- Markdown files: parse YAML front-matter for metadata, chunk body text
- PDF files: extract text with `pdf-parse`, chunk extracted text
- All chunks stored in SQLite with TF-IDF vectors as JSON

## Important Rules

1. **Stay offline** – never add features requiring internet connectivity
2. **Use the native ChatClient** – do not add the `openai` npm package; the SDK handles communication directly
3. **Use parameterised SQL** – never concatenate user input into queries
4. **Validate file paths** – ensure uploads/reads stay within allowed directories
5. **Keep the frontend framework-free** – single HTML file, inline CSS/JS
6. **Tests use `node:test`** – no external test framework dependencies
7. **Chunk sizes matter** – changes to `config.chunkSize` affect retrieval quality
8. **Temperature stays low** (0.3) – interview prep needs focused, consistent responses

## Running the Project

```bash
npm install              # Install dependencies
npm run ingest           # Index documents in docs/ into SQLite
npm run ingest -- cv.pdf # Also ingest a specific PDF
npm start                # Start web server on http://127.0.0.1:3000
npm run dev              # Start with auto-restart (development)
npm test                 # Run unit tests
```

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | /api/chat | Non-streaming chat (JSON body: `{message, history?, compact?}`) |
| POST | /api/chat/stream | Streaming chat via SSE |
| POST | /api/upload | Upload file (multipart/form-data, field: `file`) |
| GET | /api/docs | List indexed documents |
| GET | /api/health | Health check |
| GET | /api/init-status | SSE stream of model loading progress |

## Testing Guidelines

- Unit tests go in `test/*.test.js`
- Use `describe()` and `it()` from `node:test`
- Use `assert` from `node:assert/strict`
- Tests should not require Foundry Local to be running (mock or test in isolation)
- The chunker and vector store are fully testable offline

## When Modifying

- **Adding a new document type**: Add parser in `src/`, integrate in `ingest.js` and `server.js` upload handler
- **Changing the model**: Update `config.model` in `src/config.js`
- **Adjusting retrieval quality**: Tune `config.chunkSize`, `config.chunkOverlap`, and `config.topK`
- **Changing the system prompt**: Edit `src/prompts.js`
- **Adding API endpoints**: Add routes in `src/server.js`, add corresponding test in `test/server.test.js`

---
> Source: [leestott/interview-doctor-js](https://github.com/leestott/interview-doctor-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
