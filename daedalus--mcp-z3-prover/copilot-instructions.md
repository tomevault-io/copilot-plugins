## mcp-z3-prover

> MCP server exposing Z3 solver API.

# MCP Z3 Prover

MCP server exposing Z3 solver API.

## When to use this skill

Use this skill when you need to:
- Solve constraint satisfaction problems
- Check satisfiability
- Solve optimization problems

## Tools

- `create_bool_var` - Create Boolean variable
- `create_int_var` - Create Integer variable
- `create_real_var` - Create Real variable
- `create_int_constant` / `create_real_constant` - Constants
- `add_constraint` - Add constraint to solver
- `solve` - Solve current problem
- `get_model_value` - Get variable value
- `optimize` - Solve with optimization
- `reset_solver` - Reset state
- `list_variables` - List all variables

## Install

```bash
pip install mcp-z3-prover
```

---
> Source: [daedalus/mcp-z3-prover](https://github.com/daedalus/mcp-z3-prover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
