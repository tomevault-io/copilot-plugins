## expressionevaluator-2026-1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
dotnet build                                          # Build entire solution
dotnet build ExpressionEvaluator.Core                 # Build core library only
dotnet run --project ExpressionEvaluator.UI.Console   # Run console app
```

No test project exists yet. There is no linter or formatter configured.

## Architecture

This is a .NET 10 C# solution (`ExpressionEvaluator.slnx`) with three projects:

- **ExpressionEvaluator.Core** — Class library containing the expression evaluation engine. The single `Evaluator` class converts infix expressions to postfix (shunting-yard algorithm) then evaluates the postfix result. Currently only handles single-digit/single-char operands and the operators `+`, `-`, `*`, `/`, `^`, `(`, `)`.
- **ExpressionEvaluator.UI.Console** — Console app that references Core and demonstrates evaluation. Uses top-level statements.
- **ExpressionEvaluator.UI.Win** — Windows Forms app (net10.0-windows). Scaffold only — does not yet reference Core or use the Evaluator.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zulu55) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
