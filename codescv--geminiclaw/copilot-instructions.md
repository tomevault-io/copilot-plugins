## geminiclaw

> - Use `Python` as the main language.

# Technical Requirements
- Use `Python` as the main language. 
- Use `uv` to manage dependencies. Always use `uv run`, `uv add` etc instead of running Python or pip directly.
- Respect Python idioms: All Python source code should be placed under the `src` directory. The `src` directory itself should not be a Python package, and the entrypoint should be a file placed under the `src` directory.
- Unless there is a strong reason, add import statements at the top of your module.
- Use type annotations for function arguments.
- Use logging instead of print (for exceptions, line numbers etc)
- Add docstrings for all functions more than 5 lines of code.
- Add unit tests for every module you write.
- Always consult and strictly adhere to the `DESIGN.md` document throughout the entire development process. Ensure architectural alignment with the design doc before making implementation decisions. If the user asks new features or changes, always update `DESIGN.md` to be consistent.
- Always remember to update `README.md` file to reflect any feature changes that are important to the end user.
- After adding features, remember to update / run unit tests.

---
> Source: [codescv/geminiclaw](https://github.com/codescv/geminiclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
