## env

> Follow these rules when working with environment variables.

# Env Rules

- If you update environment variables, update the `.env.example` file
- All environment variables should go in `.env.local`
- Do not expose environment variables to the frontend
- Use `NEXT_PUBLIC_` prefix for environment variables that need to be accessed from the frontend
- You may import environment variables in server actions and components by using `process.env.VARIABLE_NAME`

---
> Source: [mckaywrigley/o1-pro-template-system](https://github.com/mckaywrigley/o1-pro-template-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
