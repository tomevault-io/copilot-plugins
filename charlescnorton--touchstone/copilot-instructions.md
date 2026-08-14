## touchstone

> Orientation for an AI agent (or a new contributor) working *in* this repo. The README is the user-facing

# CLAUDE.md

Orientation for an AI agent (or a new contributor) working *in* this repo. The README is the user-facing
guide (install, verbs, benchmarks, the modeled subset); this file is the map of the codebase and the rules
for changing it without breaking soundness or the gate. Read this first, and reach for a specific module
only when you touch it — you should not need to read the whole tree.

## What Touchstone is, in one paragraph

A function plus a property in, a verdict out: **PROVED** (holds for all inputs), **REFUTED** (with a
counterexample), or **UNKNOWN** (with a reason). It translates a modeled subset of Python to Z3
(corroborated by cvc5) rather than running it. A core slice — the integer IR and its weakest-precondition
generators, the division/modulo and fixed-width encodings, the interval transfers, the type-lattice join —
is machine-checked in Rocq and run as the *extracted* code; everything else is modeled soundly and
continuously cross-checked against CPython.

## Package map (`touchstone/`)

| module | responsibility |
| --- | --- |
| `core.py` | the symbolic core: the modeled-subset parser/desugar, the value lattice (`_Opaque`/`_FieldVal`/`_SafeContainer`/`_SetExpr`/`_NdArray`/`_DictParam`/`_MapVal`/…), `ev`/`symexec` (the loop-free value engine), set-operation content (`_set_binop`), strided slicing (`_slice_len`), the ord/chr codepoint bijection and bytes element values, opaque-object field modeling (`_FieldVal`), the tensor (numpy/torch) shape algebra (`_nd_method`/`_matmul`/`_torch_func`), the `math`-module and trap-free standard-library models (`_math_call`/`_fp_pow`/`_STDLIB_TF`), the loop-free heap/OO engine (`_heap_eval`, C3 MRO, method/operator dispatch), the recursive-callee contract summary (`Ctx._contract_summary`), the out-of-process sandbox, and the solver layer (`_solve`, `_solve_corro`, `solve_corroborated` confirming a PROVED with cvc5 *sequentially* -- the cvc5 binding holds the GIL through its solve, so a thread beside z3 is no real overlap and only starves the main thread). The runtime flags live here. |
| `engines.py` | the verbs (`check`, `prove`, `verify_equiv`, `verify_contracts`, `verify_change`, `scan`, …) and the deductive/CHC engines (CFG→Horn trap-freedom with per-loop invariants for nested loops, single-loop invariant synthesis, recursion incl. the array-encoded recursive-list engine, the generator-yield engine with bounded accumulator unrolling, the relational for-loop-equivalence product, interprocedural/whole-program CHC, termination, BMC, the None/Optional engines, exact bitvector). Path-sensitive definite assignment (`_use_before_def`) and the recursive-callee contract summary live here too. The module loaders (`load_module`/`load_program`/`verify_repo`). |
| `domains.py` | abstract interpretation (interval, zone, octagon, Karr, template-polyhedra, machine-integer, float-interval; each PROVED re-derived by the CHC engine via `_corroborate_domain`); the specialized theories decided in Z3 / by enumeration (concurrency: bounded interleavings + inductive + rely-guarantee over locks, counting semaphores, condition variables, async/await; IEEE-754 finiteness; separation logic via cvc5; SOS nonnegativity; sequence/string/dict laws); and the broadest trap-freedom decider (`_decide`, a cascade of `_trap_free_*` strategies) with the differential CPython oracle and the random fuzz corpora. |
| `vcgen.py` | the Rocq-extracted weakest-precondition generator (`wpg`/`vcg`), driven through `_generated/vcgen_rocq.py`, with a CPython cross-check of the Python→IR lowering; the re-checkable proof bundles (`proof_bundle`/`recheck_bundle`); and the SMTCoq obligation export (`python -m touchstone.vcgen` regenerates `proofs/touchstone_obligations.v`). |
| `inference.py` | sound (over-approximating) type inference (`infer_types`/`infer_return_type`/`infer_local_types`: a returned type-set provably contains the runtime type, else UNKNOWN) and the heuristic exact inferencer (`emit_facts`, the TypeEvalPy surface). |
| `audit.py` | `run_self_tests` (the assert suite) + the differential/soundness audits + `demo`. |
| `ci.py` | the gate: `python -m touchstone.ci` runs the self-tests, soundness audits, differential regressions, fuzz, and the committed-extraction audits. Must stay green. The bulk CPython-cross-checked phases run with `REQUIRE_CORROBORATION` off (CPython is a stronger oracle than cvc5's second opinion there); self-tests and the verification showcase keep it on, so the corroboration path stays exercised. |
| `cli.py` / `mcp.py` / `lsp.py` | the CLI verbs (one per use mode), the MCP server (verifier tools over stdio), the LSP. |
| `diagnostics.py` / `repro.py` | UNKNOWN classification + the `covers` reference; REFUTED→runnable failing test. (Re-checkable proof bundles live in `vcgen.py`.) |
| `_generated/*_rocq.py` | the Rocq extraction transpiled to Python — **do not hand-edit**; regenerate from `proofs/`. `_impl.py` re-exports the API; `__init__.py` loads lazily (z3 is imported only on first verification use). |
| `proofs/` | the Rocq + SMTCoq trust base (`touchstone_functor.v` = the VC generators sound + complete; `touchstone_encoding.v` = the `//` / `%` refinement; `touchstone_domains.v` = interval + ranking soundness; `touchstone_ndshape.v` = the tensor shape algebra; …) and the OCaml/JSON extractions. Every theorem is `Print Assumptions`-clean (no axioms, no `Admitted`). |

## How a verdict is produced

- `prove` routes by shape: a loop → `verify_chc` (Spacer invariant synthesis) → `verify_function` (a per-block Horn system that synthesizes an independent invariant per loop, so nested loops decide) → `_try_sequence_loop`, then the `_prove_via_havoc` over-approximation that decides a for-loop, comprehension, or complex assignment target (PROVED-only, never a refutation); self-recursion → `_prove_recursive_list` (a `list` parameter auto-routes to the array-encoded engine, `len(xs)` bridged) → `verify_recursive`; a class-using body → `verify_heap_property`; otherwise the Rocq-verified `prove_via_vcgen`, then `_try_sos_prove` for a polynomial nonnegativity goal (an exact-rational SOS certificate), then `verify_bitwise` for a width-bounded bitwise claim, then the typed symbolic engine — which itself falls back to `_prove_via_havoc` when it cannot translate the property. A false loop postcondition is refuted by Spacer's reachability query (through the invariant, not by unrolling to the bug's depth). A float / None claim has its own engine; `verify_equiv` decides for-loop equivalence by a relational product (both loops in lockstep, proved by one invariant).
- `check` (trap freedom, no spec) runs `verify_no_raise` (CFG→Horn) → `verify_no_raise_optional` (carries None) → `_check_trapfree_symexec` (the value engine). A loop body is over-approximated by havoc, but its *first iteration is checked exactly* (`ctx.exact_traps`: pre-loop accumulators, a freely-chosen element), so a per-element or first-iteration trap (`10 // x`, a zero accumulator on entry) refutes even under havoc. When every symbolic engine abstains on a function whose callee the inliner cannot resolve (a recursive callee), an interprocedural sandbox oracle (`_oracle_trap_refute`) is the last resort. The value engine carries a non-integer parameter rather than abandoning it: a str method (`encode`), a starred unpacking to the exact middle slice, an in-repo dataclass-style constructor (`_init_trapfree`); it strips a behavior-preserving decorator (`_is_passthrough_decorator`: `lru_cache`/`wraps`/binding markers), summarizes a constant-step for-loop counter to `s_init + step * len` (`_loop_counters`), and abstains on a possible use-before-assignment (`_definite_assignment_guard`).
- A repo callee is resolved by **inlining** (`Ctx.summary` / `inline`); a *recursive* callee makes the inliner bail (`Unsupported("recursion through …")`) — its `@ensure` contract then summarizes it at the call site (`_contract_summary`, assuming `require → ensure(result)`), else the sandbox oracle catches a trap.
- `scan(target, execute=False)` resolves a repo URL / `.py` URL / local path, runs trap-freedom triage over it, and ranks the REFUTED findings (a runtime crash above an intended `raise`). **Symbolic by default — the fetched code is never run**; `execute=True` enables the sandbox oracle. The CLI `scan` verb and the MCP `scan` tool are thin wrappers over this one function.

## Invariants you must not break

1. **Soundness over coverage.** A construct outside the modeled subset returns UNKNOWN with a reason, never a guess. PROVED is only ever for what is actually proved; an over-approximation withholds REFUTED, and sometimes PROVED (`_downgrade_overapprox`, `none_havoc`, `overapprox`). When unsure, abstain.
2. **The gate is the contract.** `python -m touchstone.ci` must pass with zero contradictions. It runs `run_self_tests` first; the assert count there auto-updates (it parses the source). Add a self-test in `audit.run_self_tests` for any new capability.
3. **Never execute analyzed code unless a flag says so.** Symbolic analysis never runs the subject. The sandbox (`core.SANDBOX_SUBJECT`, default on) and in-process (`core.ALLOW_SUBJECT_EXECUTION`, default off) paths are the only execution, used by the differential oracle and the refutation fallbacks. `scan` without `--execute` forces both off.
4. **The extracted code is generated, not authored.** Do not hand-edit `touchstone/_generated/*`. The engine runs the transpiled Rocq extraction; `committed_extraction_audit` holds each module byte-equal to the committed JSON. Change an encoding only in `proofs/` and regenerate.
5. **PROVED is corroborated.** A PROVED is confirmed by a second procedure — cvc5 where the fragment round-trips (its nonlinear coverings for polynomial goals), else a checked SOS certificate or the real-relaxation nlsat lane; the certificate records which. A cvc5 refutation of a PROVED raises `SoundnessError`. The real-relaxation lane only ever *confirms* (real-unsat proves integer-unsat); a non-integral real model is never a counterexample. Do not bypass `_solve_corro` / `solve_corroborated`.

## Dev workflow

Use a virtualenv with the pinned solver deps (`z3-solver` + `cvc5`, from `pyproject.toml`):

```sh
python -m touchstone.ci          # the gate (a few minutes); prints "CI OK" on success
python -m touchstone.examples    # one runnable example per capability, each verdict asserted
cd proofs && bash verify_all.sh  # the whole trust base across both switches; prints "TRUST BASE OK"
```

`verify_all.sh` activates each opam switch in turn -- rocq9 for the Rocq 9.0 companion files and the OCaml
extractions, certicoq-8.20 for the SMTCoq certificate check -- so one command checks everything the two
per-switch scripts (`verify_coq.sh`, `verify_smtcoq.sh`) cover separately; `proofs/Dockerfile.trustbase`
provisions both switches reproducibly. Each phase skips cleanly when its switch is absent.

- The `Constructing a fresh variable for k!…` lines on stderr are pre-existing z3/Spacer noise; filter them, do not chase them.
- Verdicts are bounded by a deterministic resource limit (`core.SOLVE_RLIMIT`), not the wall clock, so they reproduce across machines. Keep new solves under a bound rather than a timeout.

## Where to add things

- a verb or engine → `engines.py`; if it is a trap-freedom strategy, also wire it into `domains._decide`, add a `verification_benchmark` case, and a `run_self_tests` assert.
- an abstract domain → `domains.py` + a Rocq soundness lemma in `proofs/touchstone_domains.v` + a `_corroborate_domain` cross-check.
- a stdlib model → an exact / sound-over-approximation entry in `core._STDLIB`, a domain-trap-bearing function in `core._math_call`, or a pure trap-free function in `core._STDLIB_TF`, gated by a CPython audit in `audit.py` (`math_domain_audit` / `stdlib_trapfree_audit`).
- a tensor op → `core._nd_method` (a method) or `core._torch_func` (a `torch.`/`np.` function), with a shape lemma in `proofs/touchstone_ndshape.v`; element values stay opaque (shape and traps only).
- a CLI verb → `cli.py` (+ `diagnostics.capabilities` so `covers` lists it); an MCP tool → `mcp._TOOLS` + `call_tool` (+ the pinned tool-set self-test in `audit.py`).

## Pointers

- `README.md` — user-facing usage, the benchmark tables, the modeled subset, the soundness statement.
- `proofs/` — the trust base; start at `touchstone_functor.v`.

---
> Source: [CharlesCNorton/touchstone](https://github.com/CharlesCNorton/touchstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
