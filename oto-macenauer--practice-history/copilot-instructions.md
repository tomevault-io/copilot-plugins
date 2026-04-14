## practice-history

> - Static site (vanilla HTML/CSS/JS), no build step, hosted on GitHub Pages

# History Practice App

## Project Structure
- Static site (vanilla HTML/CSS/JS), no build step, hosted on GitHub Pages
- `index.html` — dashboard with test cards
- `test.html` — shared test runner page
- `css/style.css` — all styles
- `js/app.js` — dashboard logic
- `js/test-runner.js` — test engine
- `js/tests/index.js` — test registry
- `js/tests/*.js` — individual test question files

## Adding a New Test
1. Create `js/tests/your-test-id.js` exporting a `QUESTIONS` array
2. Register it in `js/tests/index.js`
3. The dashboard auto-renders from the registry

## Conventions
- All UI text in Czech
- Questions have 3 answer options, `correct` is the 0-based index of the right answer
- Each question has an `explanation` field shown after answering
- Question and option order are shuffled on each attempt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oto-macenauer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oto-macenauer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
