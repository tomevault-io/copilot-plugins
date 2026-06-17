## tiny-home-electrical-calculator

> Maintain and extend the Tiny House Electrical & Solar Calculator single-page app. Use when asked to modify calculator logic, update default appliance/circuit data, improve accessibility/UI, adjust cost or sizing assumptions, or troubleshoot behavior in index.html for this repository.


# Tiny Home Electrical Calculator Skill

Keep changes focused, readable, and beginner-friendly.

## Repository layout
- Treat `index.html` as the primary application file (HTML, CSS, and JavaScript are all in this file).
- Treat `README.md` as the user-facing project overview and usage reference.

## Core workflow
1. Read `README.md` and `index.html` before changing behavior.
2. Identify whether the request affects:
   - UI structure/styles,
   - appliance setup data,
   - calculation logic,
   - validation/error handling,
   - downloadable summary output.
3. Make the smallest safe change that satisfies the request.
4. Manually sanity-check by opening `index.html` in a browser and exercising:
   - setup selection,
   - add/clear appliance actions,
   - calculate flow,
   - summary download button.
5. If assumptions (cost, code references, solar hours, safety factors) change, update visible text so the UI stays consistent with the math.

## Implementation rules
- Preserve accessibility patterns already used (`aria-label`, `aria-live`, semantic headings, clear focus states).
- Keep formulas explicit and easy to trace; prefer named constants over magic numbers.
- Preserve NEC/safety disclaimers unless the task explicitly requests a wording change.
- For new appliance or circuit defaults, keep names descriptive and consistent with existing labels.
- Avoid large refactors unless requested; this project is intentionally simple and easy for DIY users to read.

## Validation checklist
- No JavaScript syntax errors in browser console.
- `Calculate` handles invalid/empty inputs with clear messaging.
- Result sections render correctly after changes.
- Downloaded summary reflects displayed results.
- Text claims (years, assumptions, units) match implemented calculations.

## Common edit locations in `index.html`
- Circuits catalog: `const circuits = [...]`
- Preset appliance bundles: `const typicalSetups = {...}`
- Row creation and form controls: `addApplianceRow(...)`
- Main computation pipeline: `calculate()`
- Download output: `downloadSummary()`

---
> Source: [oasiseng/Tiny-Home-Electrical-Calculator](https://github.com/oasiseng/Tiny-Home-Electrical-Calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
