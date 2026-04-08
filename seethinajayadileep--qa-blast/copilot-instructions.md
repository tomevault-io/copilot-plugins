## qa-blast

> - **TestCaseRequest**: `{ input: string, context?: string }`

# Project Constitution

## Data Schemas
- **TestCaseRequest**: `{ input: string, context?: string }`
- **TestCaseResponse**: `{ test_cases: TestCase[] }`
- **TestCase**: `{ id: string, title: string, steps: string[], expected_result: string }`

## Behavioral Rules
- User input -> Local LLM (Ollama) -> Formatted Test Cases.
- "Wow" factor in UI design (Glassmorphism, Dark Mode).
- Use `llama3.2` model via Ollama API.

## Architectural Invariants
- **Backend/Proxy**: Node.js (to handle Ollama communication and avoid CORS/formatting).
- **Frontend**: Vite + React (for Chat UI state management).
- **Styling**: Vanilla CSS (Premium aesthetic).
- **LLM**: Ollama running locally on default port (11434).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seethinajayadileep)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seethinajayadileep)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
