## tyler

> 1. Ask, don't assume. If something is unclear, ask before writing a single line. Never make silent assumptions about intent, architecture, or requirements. When running unattended, pick the most

1. Ask, don't assume. If something is unclear, ask before writing a single line. Never make silent assumptions about intent, architecture, or requirements. When running unattended, pick the most
   reasonable interpretation, proceed, and record the assumption rather than blocking.

2. Implement the simplest solution for simple problems, better solutions for harder problems. Do not over-engineer or add flexibility that isn't needed yet.

3. Don't touch unrelated code but please do surface bad code or design smells you discover with me so we can address them as a separate issue.

4. Flag uncertainty explicitly. If you're unsure about something, see point 1 above. If it makes sense to do so, conduct a small, localised and low-risk experiment and bring the hypothesis and results
   to me to discuss. Confidence without certainty causes more damage than admitting a gap.

5. I'm always open to ideas on better ways to do things. Please don't hesitate to suggest a better way, or one that has long lasting impact over a tactical change. (as a few examples)

6. Breaking changes are allowed and desirable if they lead to simpler code and architecture.

7. Do not preserve compatibility with legacy API and legacy logic when refactoring the codebase. Do not add regression tests on legacy behavior when refactoring the codebase.

8. Prefer refactoring existing tests to match new behavior instead of adding new tests. Provide explanation on every new test and explicitly ask for approval.

9. Implementation is accepted when `just ci` passes. Do not add clippy allow-s, but fix warnings and errors. 

---
> Source: [3DGI/tyler](https://github.com/3DGI/tyler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
