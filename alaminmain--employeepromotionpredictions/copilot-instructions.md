## employeepromotionpredictions

> - `PromotionAPI/`: ASP.NET Core minimal API and prediction engine (`Program.cs`, `SimulationManager.cs`).

# Repository Guidelines

## Project Structure & Module Organization
- `PromotionAPI/`: ASP.NET Core minimal API and prediction engine (`Program.cs`, `SimulationManager.cs`).
- `promotion-ui/`: React + Vite frontend (`src/`, `App.jsx`, `App.css`).
- `Promotion.db`: SQLite database used by the API.
- `EmpList.csv`, `Final_Complete_Master_List.csv`: input data files consumed by the simulation.
- `README.md`: high-level architecture and setup notes.

## Build, Test, and Development Commands
- Backend API (from `PromotionAPI/`):
  - `dotnet run --urls "http://localhost:5050"`: runs the API locally.
- Frontend (from `promotion-ui/`):
  - `npm install`: installs UI dependencies.
  - `npm run dev`: starts Vite dev server (default `http://localhost:5173`).
  - `npm run build`: production build.
  - `npm run lint`: runs ESLint.

## Coding Style & Naming Conventions
- C# uses 4-space indentation and PascalCase for public members.
- React/JSX uses 2-space indentation, camelCase for variables/functions, and PascalCase for components.
- Keep API route strings stable (`/api/...`) to avoid breaking the UI.

## Testing Guidelines
- No automated tests are currently checked in.
- If adding tests:
  - C#: create a `PromotionAPI.Tests/` project and use `dotnet test`.
  - UI: colocate tests under `promotion-ui/src/` (e.g., `App.test.jsx`) and add a test runner.

## Commit & Pull Request Guidelines
- Commit messages are short, sentence-style with context (example from history: `Initial commit: Employee Promotion Prediction System`).
- PRs should include:
  - Clear description of changes and impact.
  - Linked issue or rationale.
  - Screenshots for UI changes.

## Configuration & Data Notes
- The API expects `Promotion.db`, `EmpList.csv`, and `Final_Complete_Master_List.csv` in the repo root.
- If you relocate these files, update the paths in `PromotionAPI/Program.cs`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alaminmain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alaminmain)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
