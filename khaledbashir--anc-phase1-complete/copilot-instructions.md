## anc-phase1-complete

> **GOAL:** Identify and remove genuinely dead code. If removing something could break ANYTHING — it stays. When in doubt, leave it in.


# PROMPT — Dead Code Removal (Safe Mode)

**GOAL:** Identify and remove genuinely dead code. If removing something could break ANYTHING — it stays. When in doubt, leave it in.

## SAFETY RULES (NON-NEGOTIABLE)

1. Before deleting ANY code, you must trace every reference. Run `grep -rn "functionName" .` and `grep -rn "componentName" .` — if it appears ANYWHERE (imports, dynamic references, string interpolation, API routes, tests), it is NOT dead. Leave it.
2. If a function/component is exported but you can't find where it's consumed — it stays. It might be used by an external caller, a dynamic import, or a route you haven't traced.
3. If a file is imported anywhere via dynamic `require()`, `import()`, string-based template resolution, or Next.js magic routing — it stays.
4. If you're not 100% certain something is dead — it stays. "Probably unused" is not good enough.
5. NEVER remove commented-out code that has a TODO, FIXME, or any note explaining why it exists. That's intentional.
6. NEVER remove environment-conditional code (`if (process.env.X)`, feature flags, `NODE_ENV` checks). It may be active in production even if it's not active locally.
7. NEVER remove code that handles edge cases or error fallbacks just because you can't trigger them. They exist for a reason.
8. NEVER remove Prisma model fields, database columns, or API route parameters — these have data in production.

## WHAT COUNTS AS DEAD CODE

- Functions/components that are defined but `grep -rn` returns ZERO references outside their own file
- Imports that are imported but never used in that file (IDE gray text)
- Variables assigned but never read
- Unreachable code after unconditional `return` statements
- Duplicate function definitions where one clearly shadows the other
- Console.log/debug statements left from development

## WHAT DOES NOT COUNT AS DEAD CODE (LEAVE IT ALONE)

- Exported functions with zero internal references (could be used externally)
- Any template file in the templates directory (template selection is dynamic)
- API route handlers (Next.js routing is file-based — the file IS the reference)
- Prisma schema fields (even if no code references them — data may exist)
- CSS classes (hard to trace usage, not worth the risk)
- Type definitions and interfaces (zero runtime cost, aids readability)
- Anything in `node_modules`, `.next`, or generated files

## PROCESS

1. Run `find . -name "*.ts" -o -name "*.tsx" | grep -v node_modules | grep -v .next` to get the full file list
2. For each file, identify candidates for removal
3. For each candidate, run `grep -rn "name" . --include="*.ts" --include="*.tsx"` and paste the output
4. Only if grep returns ZERO hits outside the definition file → mark for removal
5. Collect ALL removals into a list FIRST — show me the list BEFORE deleting anything
6. Wait for my approval before executing any deletions

## OUTPUT FORMAT

For each removal, show:

```
FILE: path/to/file.ts
LINE: 45-52
CODE: [paste the actual dead code]
GREP RESULT: [paste the grep output showing zero references]
SAFE TO REMOVE: Yes — zero references found
```

If you find code that's SUSPICIOUS but you're not sure — put it in a separate "MAYBE DEAD (NOT REMOVING)" section so I can review manually.

## STANDING RULE

If any selector, path, or assumption doesn't match reality, don't stop. Read the actual source code, find the correct path, fix it, and continue. Only come back to me if you've tried twice and still can't figure it out.

## PHILOSOPHY

When in doubt, leave it in. Better to keep 10 lines of dead code than break one production feature.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khaledbashir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
