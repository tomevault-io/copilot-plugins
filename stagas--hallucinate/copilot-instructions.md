## hallucinate

> Break the task to as smaller steps possible and execute them one by one.

Break the task to as smaller steps possible and execute them one by one.
Examine all existing code carefully.
Try to integrate with existing code rather than adding new code.
Never special-case unless absolutely necessary. Instead, extend existing interfaces to support the new case elegantly.
If possible to delete or rewrite code to enable a feature, that would be better.
If a utility/hook can be extracted to be reused in multiple places it should be done so early.
Pages with similar functionality should have their internals extracted to shared utilities.
Prefer small variable names, but not abbreviations, make it readable.
Use comments only to describe the code "why", not the changes that you did.
Use modern JavaScript/TypeScript features, such as optional chaining `?.` rather than if blocks.
Trace the path of execution and analyze each step before acting.
When a problem persists, add debug logs to see what is happening.
Never use // @ts-nocheck or // @ts-ignore unless absolutely necessary.
Always use file extensions for imports.
Use Tailwindcss, for widths and heights prefer the units `dvw` and `dvh`.
Consolidate rather than duplicate. Be pragmatic.
Don't write defensive code. No defensive statements unless explicitly instructed to. No out-of-bounds checks or anything similar. When a path is ambiguous it is better to throw an error.
Never create tests and never run the tests unless explicitly said so.
Never swallow errors, either throw or print with console.error(e).
Never run the dev server yourself. Never restart the dev server.
Never overwrite a migration file unless explicitly instructed to. Always create new migrations by default, taking into account the schema so far and make sure the filename sorts last (prefix with timestamp).
Don't test if the app compiles or runs.
In Tailwind code, never use transition classes unless explicitly asked to.
Don't do defensive programming, use guards only where it is absolutely necessary. ALWAYS THROW on invalid paths. No early returns or swallowing errors.
Don't run builds yourself. Don't run `npm run build` or similar.
Never run `bun run build` or any build command.
Don't create summary documents or README documents.
Skip the summary of what you did. Just do the thing.
AssemblyScript code requires all variables to be explicitly typed and all casts done manually, i.e from f64 to f32 or i32 to f32 etc.
In AssemblyScript Map get operations required a .has() check first.
You should run code to test things using `bun -e`.

---
> Source: [stagas/hallucinate](https://github.com/stagas/hallucinate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
