## framework-conventions

> Conventions and best practices for the cursor-plugin-evals framework


# Cursor Plugin Evals — Development Conventions

## Project Structure

```
src/
├── cli/          # CLI entry point and logger
├── core/         # Types, config loader, test runner
├── docker/       # Docker health checks and setup
├── evaluators/   # Seven evaluator implementations
├── fixtures/     # Fixture recording, storage, and replay
├── layers/       # Three testing layers (unit, integration, llm)
├── mcp/          # MCP client, schema converter, tool discovery
├── reporting/    # Terminal, markdown, JSON, ES export
└── tracing/      # Span management and exporters
```

## Code Style

- ESM exclusively — all files use `import`/`export`, never `require`
- All relative imports MUST use `.js` extension (ESM requirement)
- TypeScript strict mode — no `any` without justification
- Zod v4 for schemas — `z.record()` requires TWO arguments: `z.record(z.string(), z.valueType())`
- Use `type` imports where only types are needed
- Prefer `async/await` over `.then()` chains
- Use `node:` prefix for built-in modules in new files

## Testing

- Tests use vitest with globals (no need to import describe/it/expect)
- Test files are co-located: `foo.ts` → `foo.test.ts`
- Use `vi.stubEnv()` for environment variable tests, not `process.env` mutation
- Clean up temporary files in `afterEach` blocks
- All tests should be deterministic — no dependence on live services

## Adding New Evaluators

1. Create `src/evaluators/<name>.ts` implementing the `Evaluator` interface
2. Register in `src/evaluators/index.ts` (add to `EVALUATOR_NAMES` and `createEvaluator`)
3. Add tests in `src/evaluators/<name>.test.ts`
4. Document in plugin-eval.yaml comments

## Adding New Layers

Layers are independent modules in `src/layers/<name>/`. Each exports a `run<Name>Suite()` function. Wire into `src/core/runner.ts` switch statement.

## Config Schema

When modifying the config schema in `src/core/config.ts`:
1. Update the Zod schema
2. Update the `EvalConfig` type in `src/core/types.ts` to match
3. Update plugin-eval.yaml example
4. Update the `generate` CLI command template

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
