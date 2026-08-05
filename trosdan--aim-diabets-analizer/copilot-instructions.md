## aim-diabets-analizer

> React + Vite app for analyzing diabetes CSV data using LangChain.js and Google Gemini AI. UI is in **Brazilian Portuguese**.

# Diabetes Data Analyzer — Agent Instructions

React + Vite app for analyzing diabetes CSV data using LangChain.js and Google Gemini AI. UI is in **Brazilian Portuguese**.

## Commands

```bash
npm run dev      # Dev server at http://localhost:5173/
npm run build    # Production build
npm run lint     # ESLint
npm run preview  # Preview production build
```

## Architecture

3-stage state machine in `src/App.jsx`:
```
upload → analyzed → deep
```

**Component tree:**
- `App` — stage state machine, error handling
- `CSVUploader` — API key input + CSV file upload → calls `analyzeCSVData()`
- `AnalysisResults` — renders first analysis markdown + triggers deep analysis
- `DeepAnalysisResults` — renders deep clinical analysis markdown

**AI service** (`src/services/aiService.js`):
- `setApiKey(apiKey)` — must be called before analysis (or VITE_GEMINI_API_KEY .env fallback)
- `analyzeCSVData(jsonData)` — initial stats/anomalies/patterns analysis
- `deepAnalyzeResults(firstAnalysis, jsonData)` — clinical insights/correlations
- Model: `gemini-flash-latest` via `@langchain/google-genai`, lazy-initialized
- Messages use `new HumanMessage(prompt)` — NOT object literals `{type, text}`

**CSV data:** 768 rows × 9 columns — `Pregnancies, Glucose, BloodPressure, SkinThickness, Insulin, BMI, DiabetesPedigreeFunction, Age, Outcome`

## Key Conventions

- All UI text and prompts in **Portuguese (Brazil)**
- CSS per component in `src/styles/` (e.g. `CSVUploader.css`, `AnalysisResults.css`, `DeepAnalysisResults.css`)
- Markdown output rendered with `react-markdown` + `remark-gfm` (required for table support)
- Tables in `AnalysisResults.css` styled with `#667eea` header; deep analysis variant in `DeepAnalysisResults.css`
- API key: user input takes priority over `.env`; never commit `.env` (it's in `.gitignore`)

## Known Pitfalls

- **`remark-gfm` required** for table rendering — always pass `remarkPlugins={[remarkGfm]}` to `<ReactMarkdown>`
- **Lazy model init** — `setApiKey()` resets the singleton; calling `getModel()` too early throws
- **`gemini-pro` and `gemini-1.5-flash` are deprecated** — use `gemini-flash-latest`
- **Do not initialize `ChatGoogleGenerativeAI` at module scope** — `import.meta.env` may be undefined; always use the `getModel()` lazy pattern

## References

- [README.md](./README.md) — full setup and usage guide
- [QUICK_START.md](./QUICK_START.md) — quick reference for dev setup

---
> Source: [Trosdan/aim-diabets-analizer](https://github.com/Trosdan/aim-diabets-analizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
