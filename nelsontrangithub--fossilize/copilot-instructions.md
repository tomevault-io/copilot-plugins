## fossilize

> Compress source files to AST skeletons before reading them, cutting input tokens by 70–90%. Use this skill whenever the user wants to load multiple source files into context, when context is filling up, when the user asks for "the codebase" or "the repo", or when the user mentions "fossil", "skeleton", or "compress repo". Trigger words include "fossilize", "skeleton view", "save tokens on this repo".


# 🦴 fossil — read repos as skeletons, expand on demand

## When to use

- The user asks you to load more than ~2 source files at once
- The user asks "look at the codebase" / "what does this project do" / similar
- Context window is filling and the user wants to keep working
- The user explicitly says "fossil", "skeleton", "compress"

## How to use

Instead of `Read` on each source file, run fossil first:

```
npx fossil-cli <path-or-file>
```

This prints the same code with every function/method/getter/setter body replaced by a `{ /* fossil: NL */ }` marker. Signatures, types, interfaces, class shapes, imports, and JSDoc all remain intact — which is what you need to write code that calls into the module.

### Reading a whole directory

```
npx fossil-cli src/
```

Token cost typically drops 70–90%. You'll see a summary line on stderr:
`🦴 42 files: 184,302 → 21,448 tokens (88.4% saved)`

### Just-in-time body retrieval

When you actually need to see a specific function body (the user asks you to modify it, or you need to understand exact behavior), expand only that symbol:

```
npx fossil-cli expand src/auth.ts verifyJwt
```

Returns the original full source of that function.

### Keeping certain symbols intact during the initial pass

If you know up front you'll need full bodies for specific functions:

```
npx fossil-cli src/ --keep authMiddleware,issueAccessToken
```

## When NOT to use

- Single file under ~200 lines — the marker overhead isn't worth it
- The user is debugging behavior of one specific function — read it directly
- Non-JS/TS files (Python, Go, etc.) — fossil is TS/JS/JSX/TSX only in v0.1

## What gets preserved (so you can still write correct code)

- All imports and exports
- All `interface` and `type` declarations (already compact)
- All function/method/constructor **signatures** with full parameter and return types
- All class declarations and field declarations
- JSDoc comments above declarations (unless `--strip-comments`)

## What gets removed

- Function bodies → `{ /* fossil: NL */ }`
- Method bodies → same
- Arrow function bodies (block form) → same
- Trivial 1-line bodies are **not** fossilized (the marker would be longer than the code)

## Reading a fossil output

When you see `{ /* fossil: 48L */ }` it means "the body of this function is 48 lines long and has been omitted." If you need it, run `expand`. Don't guess at the body's behavior from the signature alone if exact behavior matters — expand and read it.

---
> Source: [nelsontrangithub/fossilize](https://github.com/nelsontrangithub/fossilize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
