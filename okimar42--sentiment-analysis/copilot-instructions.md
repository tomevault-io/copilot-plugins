## sentiment-analysis

> - **Maximum File Size: 300 lines**

- **Maximum File Size: 300 lines**
  - Excluding test files and generated code
  - Split large files into smaller, focused modules

- **Refactoring Strategies**
  
  ### For React Components
  - Extract sub-components into separate files
  - Move complex logic to custom hooks
  - Separate types/interfaces into `.types.ts` files
  
  ### For API Services
  - Split by domain (auth, analysis, results)
  - Create shared types module
  - Use barrel exports for clean imports
  
  ### For Backend Views/Tasks
  - Use mixins for shared functionality
  - Create service layer for business logic
  - Split by feature area

- **Current Violations** (Reference for refactoring)
  - `backend/sentiment_analysis/tasks.py` - 1365 lines → Split into task modules
  - `backend/sentiment_analysis/test_sentiment_analysis.py` - 1553 lines → Split by feature
  - `backend/sentiment_analysis/views.py` - 431 lines → Extract view logic
  - `frontend/src/pages/AnalysisResults.tsx` - 776 lines → Extract chart components
  - `frontend/src/pages/AnalysisForm.tsx` - 370 lines → Extract form sections

- **Refactoring Example**
  ```typescript
  // Before: Large component file
  // AnalysisResults.tsx (776 lines)
  
  // After: Split into modules
  // AnalysisResults.tsx (200 lines)
  // components/charts/SentimentChart.tsx (100 lines)
  // components/charts/IQChart.tsx (100 lines)
  // components/SearchFilters.tsx (150 lines)
  // hooks/useAnalysisData.ts (100 lines)
  ```

- **Monitoring**
  - Set up pre-commit hooks to warn about large files
  - Regular code reviews should flag files approaching limit
  - Use `wc -l` or similar tools to check file sizes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okimar42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
