## zmjs

> Pure-ABAP tree-walking JavaScript interpreter. Package `$ZMJS` on SAP.

# CLAUDE.md — zmjs session context

Pure-ABAP tree-walking JavaScript interpreter. Package `$ZMJS` on SAP.
Repository: https://github.com/oisee/zmjs

## Key people

- **oisee** — owner, has admin access
- **larshp** (Lars Hvam) — collaborator with push/write access (added 2026-04-01)

## Project layout

```
src/               # abapGit-format ABAP sources (one-click SAP install)
  zcl_mjs.clas.abap          # main interpreter (~3100 LOC)
  zcl_mjs_parser.clas.abap   # parser
  zcl_mjs_env.clas.abap      # environment / scope
  zcl_mjs_arr.clas.abap      # array type
  zcl_mjs_obj.clas.abap      # object type
  zcl_mjs_test.clas.abap     # unit + integration tests
  zif_mjs.intf.abap           # shared types & constants
  zmjs_benchmark.prog.abap   # benchmark program
test262/           # 266-test ECMAScript conformance suite (JS source)
_tmp/              # local sandbox — NOT committed (see .gitignore)
  zmjs_test262.js                              # bundled test262 runner
  codex_report_2026-04-01-test262-conformance-inventory.md
cmd/               # Go CLI tools
pkg/               # Go support packages
```

## SAP objects

| Object | Description |
|--------|-------------|
| `ZCL_MJS` | Main interpreter class |
| `ZMJS_TEST262` | SA38 report — runs test262 suite from SMW0 |
| `ZMJS_TEST262.JS` | SMW0 MIME object — bundled 266-test suite |
| `ZCL_LLM_00_FILE_SMW0` | Helper: loads JS from SMW0 into string |

## Editing on SAP

Method-level edit (preferred — no full-class upload needed):

```
SAP(action="edit", target="CLAS ZCL_MJS", params={
  "method": "EVAL_NODE",
  "source": "METHOD eval_node.\n...\nENDMETHOD."
})
```

Activate after edit:
```
SAP(action="activate", target="CLAS ZCL_MJS")
```

## CI / local toolchain

```bash
# abaplint syntax check
npx @abaplint/cli .

# Transpile ABAP → JS + run (no SAP needed)
npx @abaplint/transpiler-cli
node output.mjs

# Run Go tests
go test ./...
```

GitHub Actions runs abaplint + transpile + node on every PR.

---

## test262 conformance — current state (2026-04-01)

**Score: PASS=230, FAIL=36, TOTAL=266, Time=377ms** (first ABAP run)

Full analysis: `_tmp/codex_report_2026-04-01-test262-conformance-inventory.md`

### Meta-problem: all 36 failures print `: undefined`

`new Test262Error(msg)` uses a plain function constructor (type=4).
`c_node_new` only handles type=6 (class objects) → returns `undefined`.
The harness catches `undefined` and prints `"" + undefined` = `"undefined"`.
**Fix Group A first** — it will also unmask real error messages in all other groups.

### Failure groups

| Group | Root cause | Tests | Priority |
|-------|-----------|-------|----------|
| A | `new PlainFunction()` broken | 11 | **CRITICAL** |
| B | `with` statement not implemented | 8 | low |
| C | `valueOf`/ToPrimitive coercion missing | 8 | low |
| D | Named function expression name leaks to outer scope | 3 | medium |
| E | Unicode `\uNNNN` escapes in identifier names | 2 | none |
| F | Labeled `break` from nested loop | 1 | medium |
| G | `delete` operator not implemented | 1 | low |
| H | `if (cond) IIFE; nextIIFE;` — braceless-if parser bug | 1 | medium |
| I | Calling a caught function value as plain `e()` | 1 | low |

---

## Next task: Fix Group A — `new PlainFunction()`

### Where

`zcl_mjs.clas.abap`, method `EVAL_NODE`, around line 1291.

### Current code (broken)

```abap
WHEN zif_mjs=>c_node_new.
  DATA(ls_cls) = io_env->get( <n>-str ).
  IF ls_cls-type = 6 AND ls_cls-obj IS BOUND.   " ← only handles class objects
    ...
  ENDIF.
  " plain functions (type=4) fall through → returns undefined
```

### What correct `new F(args)` must do

1. Create a fresh object: `ls_instance = object_val( )`
2. Evaluate args into `lt_new_args`
3. Call `ls_cls-fn` (the function) with `this = ls_instance`
4. If the function returns an object (type=6) → use that as result
5. Otherwise → return `ls_instance`

### Pseudocode for the fix

```abap
WHEN zif_mjs=>c_node_new.
  DATA(ls_cls) = io_env->get( <n>-str ).

  " === existing class path (unchanged) ===
  IF ls_cls-type = 6 AND ls_cls-obj IS BOUND.
    ... existing code ...
    rs_val = ls_instance.

  " === NEW: plain function constructor path ===
  ELSEIF ls_cls-type = 4 AND ls_cls-fn IS BOUND.
    DATA ls_pf_inst TYPE zif_mjs=>ty_value.
    ls_pf_inst = object_val( ).                 " fresh 'this'
    DATA lt_pf_args TYPE zif_mjs=>tt_value_slots.
    LOOP AT <n>-args INTO DATA(lr_pfa).
      APPEND eval_node( ir_node = lr_pfa io_env = io_env ) TO lt_pf_args.
    ENDLOOP.
    DATA lr_pf_inst_ref TYPE REF TO data.
    lr_pf_inst_ref = box_value( ls_pf_inst ).
    DATA(ls_ret) = call_function(
      ir_fn    = ls_cls-fn
      it_args  = lt_pf_args
      io_env   = io_env
      ir_this  = lr_pf_inst_ref ).
    " if constructor explicitly returned an object, use it
    IF ls_ret-type = 6 AND ls_ret-obj IS BOUND.
      rs_val = ls_ret.
    ELSE.
      rs_val = unbox_value( lr_pf_inst_ref ).   " return 'this'
    ENDIF.
  ENDIF.
```

### check call_function signature

Before coding, verify that `call_function` returns a value (check its signature in
`zcl_mjs.clas.abap`). If it's `RETURNING rs_val`, the pseudocode above is correct.
If it's `CHANGING`, adjust accordingly.

### Expected outcome

After fix: `new Test262Error(msg)` works → real error messages visible → can properly
diagnose remaining 25 non-A failures. Score should jump from 230 to ~241/266.

---

## Version history highlights

- **v0.3.3** — perf: slot vars + inlined value ops; fib(20)=378ms (-28%), loop10K=149ms
- **v0.3.2** — `arguments` object, slot optimization
- **v0.3.1** — function expressions, static methods, mini-runtime pattern
- **v0.3.0** — spread/rest, extends, Error, static, nullish coalescing, optional chaining
- **v0.2.x** — constructor return value, transpiled ABAP runs via open-abap-core shim

---
> Source: [oisee/zmjs](https://github.com/oisee/zmjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
