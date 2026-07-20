## class-registry

> Before writing code, check:

## Getting Started

Before writing code, check:

- `docs/plans/` — current implementation plan
- `docs/adr/INDEX.md` — prior decisions (don't re-litigate)
- `docs/future/` — deferred features (don't re-discuss)

## Architecture Decision Records

When making significant decisions — choosing between libraries, patterns, tools, or conventions — you **must** write an ADR before implementing the decision. Use the `writing-adrs` skill for the format and conventions. ADRs live in `docs/adr/`. Before writing, run `ls docs/adr/` to find the highest existing number and increment it.

If you find yourself about to establish a new cross-cutting pattern (something that will affect multiple domains or files, e.g. a testing convention, a shared utility, an error-handling approach), stop and write an ADR first even if the immediate task feels local. A pattern adopted once becomes the template for everything that follows.

## Commands

```bash
uv run autohooks activate --mode=pythonpath            # install pre-commit hook (once per clone)
uv run git commit                                      # always use instead of git commit (runs autohooks)
uv add --bounds major <package>                        # add a runtime dependency at latest version
uv add --bounds major --group dev <package>            # add a dev dependency at latest version
uv sync --group=dev                                    # sync deps after pulling
uv run pytest                                          # run tests (current Python)
uv run tox -p                                          # run tests (all supported versions)
uv run pytest --collect-only                           # verify test count (note at start of mahi; confirm it increases when done)
uv run mypy src test                                   # type check
uv run ruff check                                      # lint
uv run make -C docs clean && uv run make -C docs html  # build docs
```

**In a worktree:** the shell can silently reset to the main checkout, so always prefix state-mutating commands (`uv add`/`sync`/`run`) with `cd <worktree> &&` to ensure they hit the worktree.

## Architecture

The package is in `src/class_registry/`. The public API (`__init__.py`) exports only `ClassRegistry` and `RegistryKeyError`.

**Class hierarchy:**

- `base.BaseRegistry[ValueType, KeyType]` — read-only abstract base; defines `__getitem__`/`get`/`get_class`/`keys`/`classes`, plus extension points `gen_lookup_key` and `create_instance`.
- `base.BaseMutableRegistry[ValueType, KeyType]` — extends `BaseRegistry` with `register`/`unregister` (and deprecated `items`/`values`).
- `registry.ClassRegistry[ValueType, KeyType]` — the main concrete implementation; stores classes in a dict, supports `unique` mode to prevent key collisions.
- `registry.SortedClassRegistry[ValueType, KeyType]` — extends `ClassRegistry`; overrides `keys()` to sort by a `sort_key` attribute or callable.
- `entry_points.EntryPointClassRegistry[ValueType]` — extends `BaseRegistry`, pinning its key type to `str` (read-only); discovers classes via setuptools entry points, lazily cached.
- `cache.ClassRegistryInstanceCache[ValueType, KeyType]` — wraps a `ClassRegistry`, caching instantiated objects rather than classes (for service-registry use cases).
- `patcher.RegistryPatcher[ValueType, KeyType]` — context manager that temporarily patches a `BaseMutableRegistry`, then restores the original state on exit.

**`AutoRegister`:**
- `base.AutoRegister(registry)` — current API; returns a **base class** whose non-abstract subclasses auto-register. Requires `registry.attr_name` to be set.
- `auto_register.AutoRegister` — **deprecated** metaclass variant; kept for backwards compatibility only.

**Key design details:**
- `gen_lookup_key` is the hook for key normalisation (override in subclasses for aliases, case-folding, etc.).
- `BaseMutableRegistry._lookup_keys` maps human-readable keys → lookup keys, enabling correct iteration order and `unregister`.
- `RegistryKeyError` (subclass of `KeyError`) distinguishes registry misses from other `KeyError`s; `__missing__` is the override point for default-value behaviour.

## Docstrings

Google/Napoleon format (`Args:`, `Returns:`, `Note:`) — not Sphinx `:param:` style. Max 80 chars per line. Escape backslashes (e.g. `'\\n'` not `'\n'`). Blank line before lists inside `Args:` sections to avoid Sphinx indentation warnings. ReadTheDocs treats all Sphinx warnings as errors — resolve them before pushing.

## Tests

Every test function has a one-line docstring stating the behaviour it verifies.

## Code Comments

Place comments on the line preceding the code they document, not as trailing comments.

## Language and Style

- NZ English; incorporate Te Reo Māori where natural (e.g. "mahi", "kaupapa")
- Use "Initialises" not "Initializes"

### Writing for coding agents

- Do not document information that already exists in the coding agent's training data or could be easily discovered by reading the code.
- Do not list individual files; list high-level directories so the agent knows where to look.
- Aim for concise style that optimises token count without sacrificing clarity.

## Branches

- `main` — releases only; merge from `develop` via PR
- `develop` — main development branch
- Feature branches off `develop` for all new work

## Git Worktrees

Use the `using-git-worktrees` skill; it creates worktrees via the native `EnterWorktree` tool under `.agents/worktrees/` (gitignored). Don't hand-roll `git worktree add` when the native tool is available.

After creating a worktree, install its pre-commit hook (see the autohooks activate command in Commands). A fresh worktree has no per-worktree hooks directory, so create it first:

```bash
mkdir -p "$(git rev-parse --git-dir)/hooks"
```

---
> Source: [todofixthis/class-registry](https://github.com/todofixthis/class-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
