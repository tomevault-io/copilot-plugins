## graphk

> This file is an AI-consumable package guide for agents that need to use GraphK without re-reading the full codebase.

# GraphK Agent Guide

This file is an AI-consumable package guide for agents that need to use GraphK without re-reading the full codebase.

Scope:

- explain what GraphK is
- explain the runtime model
- explain the public package surface exported by `graphk`
- explain how to compose and execute pipelines correctly

## Purpose

GraphK is a graph-execution core for Python. It provides a small set of primitives for building executable graphs composed of nodes, pipelines, branch selectors, execution policies, and runtime sessions.

GraphK is not a full application framework. It is the execution kernel underneath graph-based workflows.

The public package is intended to be imported as:

```python
import graphk
```

## Mental Model

The core execution model is:

- `Store` is hierarchical scoped state
- `Matcher` evaluates rules against a store
- `Session` carries runtime data across execution
- `Belief` carries scoped execution state for nodes and pipelines
- `Policy` carries rule sets that validate entry and exit conditions
- `Node` is the smallest executable unit
- `Pipeline` composes nodes and nested pipelines
- `SequencePipe` executes in order
- `BranchPipe` selects one matching branch
- `MultiPipe` executes multiple matching branches
- `Runner` is the preferred end-to-end execution engine
- `Emitter` wraps a runner into a request/response API

Preferred public execution entry points:

1. `Runner` for pipeline execution
2. `Emitter` for request/response style usage

## Package Map

The package currently exports:

- `Store`
- `Matcher`
- `Logger`
- `Persistence`
- `Session`
- `Belief`
- `Policy`
- `Node`
- `Pipeline`
- `Runner`
- `Emitter`
- `BranchPipe`
- `MultiPipe`
- `SequencePipe`
- `demo`
  Demo node namespace:
  - `demo.ApplyNode`
  - `demo.IncrementNode`
  - `demo.AccumulateNode`
  - `demo.ConcatNode`
  - `demo.TextConcatNode`
  - `demo.EchoNode`
  - `demo.RouteNode`

## Scoped Key Syntax

GraphK state access is built on scoped keys. This is essential for correct use of `Store`, `Session`, and belief/policy resolution.

Supported key forms:

- `value`
  Meaning: resolve through the current store and then upward through the parent chain.

- `./value`
  Meaning: resolve only in the current store.

- `../value`
  Meaning: resolve only in the direct parent store.

- `@Scope/value`
  Meaning: resolve in the nearest store whose scope name matches `Scope`.

- `@Scope`
  Meaning: open-scope lookup used by operations like `contains()` when checking whether that scope exists in the chain.

Use `Store.get()` and `Store.set()` with these forms rather than manually walking parent links.

## Core Types

### `Store`

Purpose:

- base hierarchical key/value state container
- parent-linked
- scope-aware

Important behavior:

- inherits from `dict`
- supports scoped reads and writes
- supports flattening, saving, restoring, and cloning

Important methods:

- `rescope(scope)`
  Change the store scope name.

- `attach(parent)`
  Attach this store to a parent store.

- `detach()`
  Remove the parent link.

- `contains(key)`
  Check whether a scoped key or scope exists.

- `get(key, default=None)`
  Resolve a scoped key.

- `set(key, value)`
  Assign a value through scoped resolution rules.

- `scopes()`
  Return scope names from current store to root.

- `levels()`
  Return the number of scope levels.

- `root()`
  Return the root store.

- `flatten(top_down=False)`
  Return a resolved merged view of the store chain.

- `save()`
  Return a snapshot of local state.

- `restore(saved)`
  Restore local state from a snapshot.

- `clone(all=False)`
  Clone the store.

- `to_dict()`
  Export a resolved dictionary view.

Usage guidance:

- use `Store` as the base abstraction for hierarchical state
- use `Session` and `Belief` instead of raw `Store` when runtime semantics matter
- prefer `get()` and `set()` instead of direct dictionary access when hierarchy matters

### `Matcher`

Purpose:

- evaluate rules against a `Store`
- supports exact match, numeric comparison, wildcard, regex, callable predicates, and global predicates

Important behavior:

- inherits from `Store`
- stores rules in the same hierarchical model as other state containers

Important methods:

- `match(state)`
  Evaluate the matcher against a target `Store`.

- `to_dict()`
  Return the effective resolved rule set.

Important constants:

- matching strategy:
  - `MATCH_ALL`
  - `MATCH_ANY`

- resolver strategy:
  - `RESOLVER_BOTTOMUP`
  - `RESOLVER_TOPDOWN`
  - `RESOLVER_ROOT`
  - `RESOLVER_BOTTOM`
  - `RESOLVER_FLAT_BOTTOMUP`
  - `RESOLVER_FLAT_TOPDOWN`

Usage guidance:

- use `Matcher` directly when you need rule-based state matching
- use `Policy` when the matcher is intended to govern execution entry or exit
- `_` may be used as a global predicate key whose value is one callable or a list of callables

### `Session`

Purpose:

- runtime state flowing across nodes during execution

Important behavior:

- inherits from `Store`
- has execution status fields:
  - running
  - completed
  - cut
  - error

Important properties:

- `is_running`
- `is_completed`
- `is_cut`
- `is_error`
- `code`
- `message`

Important methods:

- `start()`
  Mark the session as running.

- `completed()`
  Mark the session as completed.

- `step()`
  Advance the session running code.

- `cut()`
  Mark the session as cut.

- `error(msg, code=404)`
  Mark the session as failed.

- `to_dict()`
  Export session state including status metadata.

Usage guidance:

- treat `Session` as the single source of execution result state
- read final results from the session after `Runner.run()` or `Emitter.request()`
- use `to_dict()` for inspection and serialization-friendly output

### `Belief`

Purpose:

- scoped execution state attached to nodes, pipelines, or runners

Important behavior:

- inherits from `Store`
- usually attached into a parent chain rather than used alone

Usage guidance:

- use `Belief` for scoped execution state such as stage names, routing values, modes, or environment-like runtime settings
- do not use global module variables when scoped belief can express the same information

### `Policy`

Purpose:

- execution-aware matcher used for entry and exit validation

Important behavior:

- inherits from `Matcher`
- carries an execution strategy

Important property:

- `execution_strategy`

Important constants:

- `EXECUTION_STOP`
- `EXECUTION_SKIP`
- `EXECUTION_RECONSIDER`

Usage guidance:

- attach `entry_policy` and `exit_policy` to nodes or pipelines
- use `EXECUTION_STOP` when a failure should terminate execution
- use `EXECUTION_SKIP` when a failure should skip the affected node and continue

### `Node`

Purpose:

- abstract executable unit in the graph

Required methods for subclasses:

- `ping()`
- `info()`
- `step()`

Important fields:

- `scope`
- `belief`
- `entry_policy`
- `exit_policy`
- `parent`
- `session`

Important methods:

- `attach(parent)`
  Attach node belief and policies to a parent node.

- `detach()`
  Remove parent linkage.

- `start(session)`
  Bind node to a session.

- `complete()`
  Unbind node from the session.

Usage guidance:

- subclass `Node` to create custom execution behavior
- implement `step()` as a generator
- write results into the active `session`
- `info()` should at least expose a stable human-readable name

### `Pipeline`

Purpose:

- abstract composite node that coordinates traversal across child nodes

Important behavior:

- inherits from `Node`
- contains child nodes or nested pipelines
- tracks `current` node and traversal iterator

Important methods:

- `add(node, index=None)`
  Insert a child node.

- `start(session)`
  Attach the pipeline to a session and reset traversal state.

- `complete()`
  Complete the pipeline and detach session state.

- `step()`
  Execute one step of the current child node.

- `sequence()`
  Return the resolved ordered node sequence.

- `forward()`
  Abstract traversal method implemented by subclasses.

Usage guidance:

- use `Pipeline` as a base class only
- use concrete subclasses such as `SequencePipe`, `BranchPipe`, or `MultiPipe` for execution

### `SequencePipe`

Purpose:

- execute nodes and nested pipelines in sequential order

Important method:

- `forward()`
  Yields nodes in execution order, flattening nested pipelines during traversal

Usage guidance:

- use `SequencePipe` as the default pipeline type
- use it both for simple flat pipelines and nested composition

### `BranchPipe`

Purpose:

- select exactly one matching branch pipeline for execution

Important fields:

- `tests`
- `selection_policy`
- `selected_index`
- `selected`

Important constants:

- `SELECT_FIRST`
- `SELECT_RANDOM`

Important methods:

- `add(node, test=None, index=None)`
  Add a branch pipeline and optional matcher test.

- `info()`
  Return branch metadata including selected branch index.

- `forward()`
  Yield nodes only from the selected matching branch.

Usage guidance:

- every branch node should itself be a `Pipeline`
- `tests` and `nodes` should align by index
- use `SELECT_FIRST` for deterministic routing
- use `SELECT_RANDOM` only when nondeterministic or seeded selection is intended

### `MultiPipe`

Purpose:

- execute all matching branch pipelines in a selected order

Important fields:

- `order_policy`
- `selected_indices`
- `selected`

Important constants:

- `ORDER_STORED`
- `ORDER_RANDOM`

Important methods:

- `info()`
  Return metadata for multi-branch execution.

- `forward()`
  Yield nodes from all matching branches.

Usage guidance:

- use `MultiPipe` when multiple routes should execute rather than only one
- use `ORDER_STORED` for stable execution order
- use `ORDER_RANDOM` only when explicitly desired

### `Runner`

Purpose:

- preferred execution engine for pipelines

Important behavior:

- inherits from `Node`
- owns the pipeline execution lifecycle
- handles entry and exit policy checks
- manages rollback and session failure behavior

Important methods:

- `enable_log(level=Logger.BASIC)`
  Enable structured execution logging.

- `start()`
  Prepare the runner and pipeline for execution.

- `step()`
  Execute one runner step.

- `run(until=-1)`
  Execute until completion or until a step limit is reached.

- `info()`
  Return runner metadata.

Usage guidance:

- prefer `Runner` over manual node stepping for full pipeline execution
- create it with `Runner(pipeline, session)`
- call `start(...)` before `run()` or `step()`
- read results from the session after completion

### `Emitter`

Purpose:

- stateful request/response wrapper around `Runner`

Important behavior:

- holds one program reference and one session manager
- caches the last returned session in `response`

Important methods:

- `request(id=None, payload=None, context=None)`
  Resolve a session through the manager, resolve a pipeline through the program, run it, and return the resulting session.

- `enable_log(level=Logger.DEBUG)`
  Enable logging through the wrapped runner.

Usage guidance:

- use `Emitter` when the package should feel like a reusable request/response component rather than a manually managed runner
- prefer it for service-like or workflow-like interfaces

### `Logger`

Purpose:

- central package logging contract

Important constants:

- `SILENT`
- `ERROR`
- `BASIC`
- `DETAIL`
- `DEBUG`

Important methods:

- `enable(level=BASIC)`
- `format_event(...)`
- `log(...)`

Usage guidance:

- use `Logger` for shared package logging levels
- use `Runner.enable_log()` or `Emitter.enable_log()` rather than wiring logging manually in normal execution flows

### `Persistence`

Purpose:

- save and load `Session` snapshots from disk

Important methods:

- `save(session, target)`
  Save a session to a file or directory target.

- `load(source)`
  Load a session from disk.

Usage guidance:

- use `Persistence.save()` for snapshotting runtime state
- use `Persistence.load()` to restore a saved session for inspection or continued processing

## Demo Nodes

These are reusable convenience nodes exposed through `graphk.demo`.

### `ApplyNode`

- apply one callable to the active session
- optionally store the result in a target key and/or response key

### `IncrementNode`

- increment one numeric session field
- optionally append execution order
- optionally write a response payload

Alias:

- `AccumulateNode = IncrementNode`

### `ConcatNode`

- append text into a session string field
- optionally write the final text as the response

Alias:

- `TextConcatNode = ConcatNode`

### `EchoNode`

- read one session key
- write a simple echo response containing the value and its doubled form

### `RouteNode`

- write the selected route name into the session
- useful in branch-routing demos or simple route handlers

## Common Lifecycles

Demo node import pattern:

```python
import graphk


node = graphk.demo.IncrementNode("A")
```

### Preferred End-To-End Execution

```python
import graphk

pipeline = graphk.SequencePipe(nodes=[...])
session = graphk.Session(...)
runner = graphk.Runner(pipeline, session)

runner.start()
runner.run()

result = session.to_dict()
```

Use this flow by default.

### Request/Response Execution

```python
import graphk

program = graphk.Program({"main": graphk.SequencePipe(nodes=[...])})
emitter = graphk.Emitter(program)

session = emitter.request("main", payload={"value": 10})
response = emitter.response
```

Use this when the caller thinks in terms of requests and responses.

### Manual Traversal

```python
import graphk

pipeline = graphk.SequencePipe(nodes=[...])
session = graphk.Session(...)
session.start()
pipeline.start(session)

for node in pipeline:
    node.start(session)
    while True:
        try:
            pipeline.step()
        except StopIteration:
            node.complete()
            break

pipeline.complete()
```

Use this only when you explicitly need low-level execution control.

## Preferred Usage Rules

- use `Runner` for normal execution
- use `Emitter` for request/response interfaces
- use `SequencePipe` unless you specifically need branching behavior
- use `BranchPipe` when exactly one branch should execute
- use `MultiPipe` when multiple matching branches should execute
- write node outputs into `session`
- use `Belief` for scoped execution settings
- use `Policy` for entry/exit validation
- read final state from `session.to_dict()`

## Common Mistakes

- Do not bypass `Runner` unless you intentionally want manual control.
- Do not treat `Belief` as the final result store; final runtime output belongs in `Session`.
- Do not use direct dictionary access when scoped resolution matters; use `get()` and `set()`.
- Do not put non-pipeline nodes directly into `BranchPipe` branches; branch items should be pipelines.
- Do not assume branch tests are evaluated globally; they resolve against the active hierarchical state.
- Do not reimplement request/response orchestration when `Emitter` already matches the use case.

## Learn-By-Example Sources

The repository test suite already acts as practical usage documentation. Good starting points are:

- `tests/core/test_sequence_1.py`
- `tests/core/test_branch_1.py`
- `tests/core/test_multi_1.py`
- `tests/core/test_runner_1.py`
- `tests/common/test_store_1.py`
- `tests/common/test_matcher_1.py`

## Editing Rules For Agents

When editing this package:

- preserve module header structure
- preserve section separators
- preserve docstrings and comments unless explicitly asked to change them
- follow the project coding style:
  - fail-fast guards first
  - deliberation second
  - optional early returns only in guards
  - one terminal return at the end of deliberation

---
> Source: [kochf1/graphk](https://github.com/kochf1/graphk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
