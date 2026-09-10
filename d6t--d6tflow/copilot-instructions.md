## d6tflow

> Guidance for working in the **d6tflow** repo.

# CLAUDE.md

Guidance for working in the **d6tflow** repo.

## What this is

d6tflow is a small, self-contained Python library for building data-science workflows: you
declare `Task` classes with parameters, dependencies (`requires`), and a `run()` that
`save()`s output; the engine runs the DAG in dependency order and skips tasks whose output
already exists. It has no heavyweight workflow-engine dependency — the task model, parameters,
and executor all live in `d6tflow/core.py` + `d6tflow/parameter.py`.

## Layout

```
d6tflow/
  __init__.py        # public API: run, preview, Workflow, WorkflowMulti, FlowExport/Import,
                     #   invalidate_*, requires/inherits, re-exported Parameter types
  core.py            # the engine: Task, Register metaclass, Target/LocalTarget, flatten,
                     #   getpaths, task_id_str, inherits/requires, find_deps, build()
  parameter.py       # Parameter, Int/Float/Bool/Date/Dict/List/Enum Parameter
  tasks/__init__.py  # TaskData (+ TaskCache/Json/Pickle/CSV/Excel/Pq/Markdown...), TaskAggregator
  targets/__init__.py# CacheTarget, _LocalPathTarget, DataTarget + format targets, Target re-export
  settings.py        # global settings (dirpath, cached, check_dependencies, task-id lengths...)
  cache.py           # `data` = in-memory target cache (dict)
  utils.py           # print_tree (preview), traverse, param generators, bcolors
  functional.py      # decorator-based functional Workflow API (uses only d6tflow.requires)
tests/               # pytest suite (see tests/setup.md)
docs/                # examples + docs/todo/ design notes
```

Ignore `bak/`, `build/`, `dist/`, `*.egg-info/`, `data/`, `models/`, `tests-data/` — local
artifacts / backups, not source. (`d6tflow/core/` is an empty stale dir; the real module is
`d6tflow/core.py`.)

## Public API (what exists — check here before adding new code)

Everything below is exported from `d6tflow` (top-level) unless noted. Reach for these
instead of reinventing them.

**Run / load**
- `run(tasks, forced=None, forced_all=False, forced_all_upstream=False, confirm=False, abort=True, ...)` — run task(s) in dependency order (`workers` is accepted but ignored — sequential engine).
- `preview(tasks, show_params=True, ...)` / `show(task)` — print the execution tree without running.
- `runLoad(task, params=None, load=True, taskLoad=None, reset=False)` — one-liner: build a `Workflow`, optionally `reset`, `run`, then `outputLoad` and return it. `runIt(...)` is `runLoad(..., load=False)`. `runIterConcat(...)` is the same shape (experimental). Prefer these for quick run-and-fetch over hand-rolling a `Workflow`.

**Workflow objects** — `Workflow(task=None, params=None, path=None, env=None)` and `WorkflowMulti(...)` (multi-experiment: `params` is `{flow_name: {param: val}}`, methods take an optional `flow=` selector). Key methods on both: `run`, `preview`, `complete`, `outputLoad` / `outputLoadAll` / `outputLoadMeta` / `outputLoadMetaJson`, `outputPath`, `reset` / `reset_upstream` / `reset_downstream`, `set_default`, `get_task`, `attach_flow`.

**Deps / params (decorators)** — `@requires(*tasks | {name: task})` copies parent params **and** wires `requires()`; `@inherits(...)` copies params only (adds `clone_parent`/`clone_parents`, you write `requires()` yourself). Params live in `parameter.py`, re-exported: `Parameter`, `IntParameter`, `FloatParameter`, `BoolParameter`, `DateParameter`, `DictParameter`, `ListParameter`, `EnumParameter` (use `significant=False` to exclude from `task_id`).

**Share / move flows** — `FlowExport(tasks=None, flows=None, save=False, path_export='tasks_export.py')` generates standalone task files; `FlowImport(...)` loads them back. (Generated-text contract — see the FlowExport note under Conventions.)

**Invalidate** — `invalidate_upstream(task, confirm=False)`, `invalidate_downstream(task, task_downstream, confirm=False)`, plus `taskflow_upstream` / `taskflow_downstream`. NB: `invalidate_all` and `invalidate_orphans` are **stubs that raise `NotImplementedError`** — don't point users at them.

**Config** — `set_dir(dir=None)` (init + set data dir), `enable_cloud_storage(protocol, bucket, prefix=None)` / `enable_gcs(bucket, prefix=None)` (fsspec-backed; needs the `gcs`/`s3`/`cloud-base` install extra), `enable_logging()` / `disable_logging()`, and the mutable `settings.*` (`dirpath`, `cached`, `check_dependencies`, `execution_summary`, task-id lengths).

**Functional API** — `d6tflow.functional.Workflow` is a separate decorator-based style (`@flow.task`, `@flow.requires`, `@flow.params`, `@flow.persists`). Independent of the class-based API above; don't mix the two in one example.

**Task-body idiom** (inside `run()`): load upstream with `self.inputLoad()` (single, or `a, b = self.inputLoad()` for multiple; `self.input()[key].load()` only to select one named/indexed input or a specific `persist=` output); save with `self.save(...)` / `self.saveMeta(...)`. Keep examples on `self.inputLoad()` — the docs standardize on it.

## Architecture notes that bite

Read `docs/todo/20260606-sys-decouple-luigi.md` and
`docs/todo/20260606-sys-param-global.md` before changing the engine — they capture the
non-obvious decisions. Highlights:

- **Sequential engine.** `core.build()` runs the DAG in-process, in dependency order. The
  `workers` argument is accepted but ignored. It handles `external=True` tasks (never run),
  generator `run()` (TaskAggregator / dynamic yields), errors (mark failed, record first
  exception, `d6tflow.run(abort=True)` raises `RuntimeError` *chained* to it — see "Logging"),
  and is re-entrant (a task's `run()` may call `d6tflow.run()` — flow-within-a-flow).

- **Deterministic `task_id`**: `f"{family}_{summary}_{md5(sorted_params_json)[:10]}"`. Many
  tests hard-code ids like `Task1__99914b932b`. If you touch `task_id_str`, expect those to
  break — keep the algorithm stable unless intentionally rebaselining.
  `task_id.split('_')[0]` must equal the task family (directory convention in `_getpath`).

- **Instance memoization is load-bearing.** `Register.__call__` (core.py) caches instances by
  `(class, serialized-params)`, so `Task(**same_params)` returns the *same* object and `__init__`
  runs only on the first call. `Workflow` relies on this: it sets per-flow `path`/`flows` by
  *mutating* a task instance, then retrieves the same instance later via `outputPath`/`FlowExport`.
  `path`/`flows` are NOT Parameters, so they don't ride through `clone()` — the cache is the only
  thing that carries them to upstream tasks. Don't remove it without redesigning that propagation.

- **In-memory cache + mutation gotcha.** `TaskCache`/`CacheTarget.load()` returns the cached
  object *by reference*. Mutating a loaded input in place corrupts upstream cached data. This is
  expected behavior, not a bug.

- **Params.** Only the trimmed set in `parameter.py` exists (`Parameter`, `Int/Float/Bool/Date/
  Dict/List/Enum`). `significant=False` is excluded from `task_id` (so two tasks differing only
  in an insignificant param share an id but are distinct cached instances). Dict/List values are
  stored raw (not frozen) and serialized with sorted keys for id determinism.

- **Stable contract for d6tflow2** (a separate downstream repo): keep `tasks.TaskData`/
  `TaskPqPandas`/`TaskAggregator`, `targets.DataTarget`/`_LocalPathTarget`/`Target`,
  `Task.to_str_params(only_significant=…)`, `clone`, `get_params`, `task_id`/`task_family`,
  and `external`/`persist` semantics stable.

## Logging (dev notes)

loguru-based, designed to a plan: `docs/todo/20260606-sys-logging.md` (read it before changing
logging). User-facing docs: `docs/source/logging.rst`. Key facts for working on it:

- **`d6tflow/log.py` owns everything.** It holds the single `logger`, calls
  `logger.disable("d6tflow")` at import (library pattern: silent until the app opts in), and
  exposes `enable_logging(level=None, sink=sys.stderr)` / `disable_logging()` (re-exported from
  `__init__.py` as `d6tflow.enable_logging` etc.). Every other module does
  `from d6tflow.log import logger` so records stay in the `d6tflow.*` namespace.
- **Namespace gating is by *caller module name*, not the logger object.** `logger.disable/enable`
  and the sink's `filter="d6tflow"` both key off `record["name"]`, which loguru derives from the
  emitting frame. That's why engine logs (emitted inside `core.py`/`tasks/__init__.py`) are
  governed correctly, but a task author's `self.logger.info()` — emitted from *their* module —
  would NOT be. Hence `TaskLogger` (in `log.py`): a thin facade that `__getattr__`-delegates
  every loguru method but wraps the call in a closure *defined in log.py*, so the actual loguru
  call's frame name is `d6tflow.*` and the record is gated like the rest. It also `logger.patch`es
  the display name to `d6tflow.task` and pre-binds `task_id`/`task_family` into `extra`
  (`bind()` returns a `TaskLogger` so the wrapping survives added context). `Task.logger`
  (core.py) returns a cached `TaskLogger`. **Don't** replace it with a plain `logger.bind(...)`
  or have `__getattr__` return the loguru method directly — then the emit runs in the *caller's*
  frame, gating silently breaks (records leak via any catch-all handler and ignore the on/off
  switch).
- **`enable_logging` removes loguru's default handler (id 0).** loguru ships a pristine
  unfiltered stderr handler at DEBUG; without removing it, enabling would (a) double-print every
  record and (b) ignore `level=` (id 0 shows DEBUG regardless). So with the default
  `sink=stderr`, `enable_logging` drops its previously-added sink *and* handler 0, then adds one
  filtered handler. `sink=None` touches no handlers (for apps that configured their own loguru).
  Module global `_handler_id` tracks the sink so repeat calls replace instead of stack.
- **Default level** comes from `settings.log_level` (`'INFO'`) via a *lazy* import inside
  `enable_logging` (top-level `log.py`→`settings` would be circular: settings→core→log).
- **Failure path** (see plan §3b): `build()` no longer does `traceback.print_exc()`; it logs the
  traceback via `logger.opt(exception=True).error(...)` and records the first exception on
  `RunResult.first_exception`; `__init__.py:run()` does `raise RuntimeError(...) from
  result.first_exception` so the propagated stack is one connected chain.
- **Level taxonomy:** INFO = task start/complete(+duration)/failure/run-summary/invalidation;
  DEBUG = cached-skips + save/load/input I/O (keys) + generator yields; WARNING = external task
  missing output; ERROR = task `run()` raised. Keep routine I/O at DEBUG so default INFO stays
  quiet.
- **Tests are unaffected** because logging is disabled by default and tests don't capture
  stderr — adding log points won't move the 73-passing baseline. To assert on log output, attach
  a loguru sink that appends to a list (loguru doesn't use stdlib `logging`, so pytest `caplog`
  won't see it).

## Plans (design notes you can execute from a clean session)

Non-trivial work is planned first, and the plan is saved **in the repo** at
`docs/todo/<YYYYMMDD>-<area>-<topic>.md` (e.g. `docs/todo/20260606-sys-logging.md`,
`20260606-sys-decouple-luigi.md`). `<area>` is a short tag like `sys`, `engine`, `tasks`.

These plans double as the architecture record (the "Architecture notes that bite" section above
points at them) **and** as executable specs. The intended workflow is: write the plan → clear
Claude Code (`/clear`) → in the fresh session say "execute `docs/todo/<file>.md`". Because the
new session has **no memory of the planning conversation**, the plan file must be completely
self-contained.

A plan file MUST contain, in order:

1. **`## Context` — the WHY.** The problem or need, what prompted it, and the intended outcome.
   Write it so someone with zero prior context understands *why this is worth doing* before any
   how. Include the current broken/limiting behavior (quote real output/errors where it helps).
2. **`### Design decisions`** — the choices made and *why* (and notably what was rejected), so the
   executor doesn't relitigate them. Mark anything the user explicitly confirmed.
3. **`## Implementation`** — numbered, ordered steps. Each step names the **exact file** and the
   function/seam (e.g. ``core.py:493`` `build()` before `task.run()`), shows the code or precise
   change, and is concrete enough to apply without guessing. For a pattern repeated across files,
   describe it once and list representative paths.
4. **`## Files modified`** — the full list, one line each, with what changes in each.
5. **`## Verification`** — exactly how to prove it works end-to-end: commands to run, expected
   output, and the test baseline to hold (see "Running tests" — currently **73 passing**).

Keep it scannable but complete: enough that a clean session can execute it faithfully, including
re-deriving the goal. Don't reference "the conversation" or "as discussed" — inline everything.

**When a plan is implemented:**

- Leave the file in `docs/todo/` as the design record.
- If the implementation diverged from the plan (different approach, a fix the plan didn't
  anticipate, a rejected step), append an **`## Implementation notes (divergences from the plan
  as built)`** section to the plan file capturing *what* changed and *why* — so the file stays a
  truthful design record, not a stale spec. `docs/todo/20260606-sys-logging.md` is the worked
  example (its addendum records the `TaskLogger` facade and the default-handler removal that the
  original plan missed).
- **Commit the plan file in the same commit as the code it describes** (and its divergence notes
  with the code that caused them), so the design record and the implementation never drift apart
  in history.

## Running tests

From the repo root (paths resolve `data/` → `tests/data/`):

```bash
python -m pytest tests/test_main.py tests/test_workflow.py \
    tests/test_workflowMulti.py tests/test_workflowMulti2.py -q
```

Only `test_*.py` are collected (see `tests/setup.md` for what each covers). A benign
`UserWarning: datatable failed` and sklearn convergence warnings are expected. Current
baseline: **73 passing**. Needs `pandas`, `pyarrow`, `openpyxl`, `scikit-learn`, `jinja2`,
`tables`; `datatable` is optional (soft-fails).

## Conventions

- Platform is Windows (PowerShell); tests compare against `pathlib.Path(...)` (not raw strings)
  so they pass cross-OS. Keep new path assertions OS-agnostic.
- Match surrounding style; the codebase is plain, comment-light, no type annotations.
- When changing `FlowExport`'s generated-file template, update the exact expected strings in
  `tests/test_workflow.py::TestFlowExports` (they assert generated text byte-for-byte).
- Don't commit/push unless asked. Default branch is `main`; feature work is on `decouple*`.
- **Multi-line strings: match the here-string syntax to the shell tool you're actually calling.**
  The Bash and PowerShell tools use different syntaxes — don't mix them. For a multi-line commit
  message: in the **Bash** tool use a quoted heredoc (`git commit -F - <<'EOF' … EOF`, or
  `-m $'line1\nline2'`); in the **PowerShell** tool use a here-string (`@'…'@`, closing `'@` at
  column 0). Crossing them silently corrupts the message (e.g. `@'…'@` in bash leaks literal `@`
  lines). When unsure, write the message to a temp file and `git commit -F`.

---
> Source: [d6t/d6tflow](https://github.com/d6t/d6tflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
