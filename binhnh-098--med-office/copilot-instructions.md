## med-office

> - Write code that is easy to read, edit, and extend.

# Coding Rules for Agent

## 1. Code Structure

- Write code that is easy to read, edit, and extend.
- Split logic into small functions/components; each part should do one clear job.
- Avoid overly long files. If a file starts growing too large, split it into modules.
- Prefer clear structure: `components`, `hooks`, `services`, `utils`, `types`, `constants`.

## 2. Naming

- Use clear names for variables, functions, and components.
- Avoid unclear abbreviations.
- Boolean names should start with `is`, `has`, `can`, or `should`.
- Function names should use verbs, for example `getUser`, `createOrder`, `validateForm`.

## 3. Clean Code

- Do not leave dead code, debug `console.log`, or unnecessary comments.
- Avoid duplicated code. If logic is repeated two or more times, consider extracting it.
- Do not hard-code important values; move them to constants/config.
- Prefer early returns to reduce nested `if/else`.

## 4. Maintainability

- Keep changes scoped to the requested work.
- Do not perform large refactors unless requested.
- Do not change public APIs, props, or database schema unless needed.
- Preserve backward compatibility whenever possible.

## 5. Error Handling

- Always handle errors in API calls, form submit handlers, and async functions.
- Error messages should be clear and understandable.
- Do not silently swallow errors.
- For frontend work, always include loading, empty, and error states.

## 6. Type Safety

- Prefer clear TypeScript `type`/`interface` definitions.
- Avoid `any` unless it is truly necessary.
- Move shared types into separate files.
- Validate input data when it comes from users or APIs.

## 7. UI/UX

- UI components must be responsive.
- Do not mix too much business logic into render components.
- Forms need validation and disabled states while loading.
- Dangerous actions should require confirmation when appropriate.

## 8. API & Data

- Put API calls in dedicated service files.
- Do not scatter direct API calls across many components.
- Standardize response/error handling when possible.
- Do not expose secrets, tokens, or API keys in frontend code.

## 9. Testing / Verification

- After changes, check compile/lint errors when possible.
- For important logic, add tests or describe the checked cases.
- Do not claim tests were run if they were not actually run.

## 10. Output Expectation

- Read existing context carefully before coding.
- After changes, briefly explain what changed.
- If issues outside the requested scope are found, note them separately and do not make large unrelated changes.
- Final code should be runnable, clean, and consistent with the current project style.

---
> Source: [binhnh-098/med_office](https://github.com/binhnh-098/med_office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
