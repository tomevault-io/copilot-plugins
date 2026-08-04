## alpagym

> These rules apply to ALL newly written or modified code. Follow them even when

# AlpaGym — AI Agent Context

## Coding Principles

These rules apply to ALL newly written or modified code. Follow them even when
surrounding code uses different patterns.

### Rationale

We aim for a *lightweight* codebase that is easy to read, understand and modify.
Our target audience are researchers with their unique ideas and setup who will
likely not use the code exactly as is, but will need to understand and modify it.

### Guidelines

- **Readability over flexibility.** Introduce abstractions only when they reduce
  net complexity (the abstraction is simpler to understand than the in-lined
  code). Don't prematurely introduce them (YAGNI).
- **Minimal set of features.** Avoid bloat and feature creep, instead focusing
  on the core functionality. We also don't aim to cover all eventualities and
  edge cases. Solve the present problem, not a hypothetical future one.
- **One way of doings things.** Be opinionated and aggressively remove legacy
  code paths. Avoid special cases. Even for different configurations (e.g.
  different drivers or deployments), try to unify APIs as much as possible.
- **Locality of behavior.** Avoid indirections, one-line wrapper functions,
  gratuitous delegation, and unnecessary class hierarchies. Every "hop" must
  earn its keep by improving readability or enabling genuine reuse. For example,
  when extracting code into a function, ask whether it is easier to understand
  the abstraction (e.g. function name) or the inlined code. If the latter, keep
  it inlined. In this case, it is likely also not critical to be tested in
  isolation.
- **Configuration in one place.** Single source of truth for all config
  (proposed: YAML + Hydra). No env-variable overrides, no parallel CLI args, no
  `constants.py` defaults that shadow the real config.
- **Method docstrings.** All methods should have docstrings. Public methods
  should have detailed docstrings that describe their behavior, arguments, and
  return values. Private methods should still be documented, but the level of
  detail should match the method's complexity.
- **Google Python Style Guide** as general baseline.

### Rules

- Avoid `getattr` + raise-on-empty blocks for defensively accessing attributes.
- In docstrings, prefer simple, clear explanations over jargon-rich descriptions.
- Explicit local call signatures: Do not introduce dataclasses that only
  bundle arguments for a local method or helper. Prefer explicit keyword
  parameters so the callee signature shows what it needs. Use dataclasses for
  real domain objects, protocol payloads, configuration schemas, queue messages,
  and persisted artifacts, not for synthetic `FooRequest` or `FooArgs` argument
  bags.
- Explicit local types where inference is weak: Annotate local variables
  when assigning the return value of an untyped, loosely typed, generated,
  factory, or boundary-crossing call, especially when the concrete type is
  important for readability or static checking. Apply this only when the
  annotation is lightweight and names a useful concrete type. Do not add
  annotations that require extra `cast(...)` calls, spell only `object` or
  similarly broad types, duplicate types already clear from a constructor or
  strongly typed helper, or otherwise make local code harder to read.

### Examples

#### Error Handling

- Fail fast, never silently fall back
- Raise an error on unexpected input. Do not default-away problems. Rely on
  default exceptions (e.g. `KeyError`) rather than catching and re-raising with
  a custom message.

```python
# DISCOURAGED -- silent fallback hides bugs
def get_backend(name: str) -> Backend:
    return BACKENDS.get(name, DefaultBackend())

# DISCOURAGED -- fails fast, but with an unnecessary wrapper that adds no value
def get_backend(name: str) -> Backend:
    if name not in BACKENDS:
        raise ValueError(f"Unknown backend: {name!r}")
    return BACKENDS[name]

# PREFERRED -- fail fast and minimize abstraction layers
backend = BACKENDS[name]
```

#### Minimize abstraction layers

Add a new class/wrapper/layer only when it removes duplication across 3+
call sites. One caller = inline the logic.

```python
# DISCOURAGED -- unnecessary wrapper for a single use
class QueryExecutor:
    def __init__(self, db):
        self.db = db
    def run(self, sql):
        return self.db.execute(sql)

# PREFERRED -- call directly
result = db.execute(sql)
```

#### Minimize the use of constants

Inline literals by default. Prefer reading the real value at the call site over
chasing named constants. Do not introduce constants, constant classes, or
dataclass “namespaces” for file-local strings, numbers, prefixes, or small path
fragments. This is stricter than the general repo baseline: reuse alone does
not justify extracting a constant in this project.

Introduce a named constant only in rare cases where multiple components must
agree on the exact same value and inlining it in each place would risk hard to
diagnose/detect drift. Typical examples are env var names or schema keys shared
by producer and consumer, and file or directory names used across multiple
components. A value being part of an external interface is not, by itself,
enough to justify a constant.

If a constant appears necessary to keep internal modules in sync, first ask
whether the code should be reorganized so one module owns that responsibility.
Often the better fix is to centralize the behavior, eliminating the need for the
constant entirely.

```python
# DISCOURAGED -- unnecessary constants
@dataclass(frozen=True)
class ConfigKeys:
    CUSTOM: str = "custom"
    REWARD_IMPL: str = "reward_impl"


CONFIG_KEYS = ConfigKeys()

raw = _read_required_config_value(
    config,
    [CONFIG_KEYS.CUSTOM, CONFIG_KEYS.REWARD_IMPL],
)

# PREFERRED -- inline literals
raw = _read_required_config_value(config, ["custom", "reward_impl"])
```

```python
# DISCOURAGED -- unnecessary constant for a single-use seed offset
@dataclass(frozen=True)
class MetricsGroupRolloutSeedDefaults:
    SEED_OFFSET: int = 1


METRICS_GROUP_SEED_DEFAULTS = MetricsGroupRolloutSeedDefaults()

return base_seed + group_idx + METRICS_GROUP_SEED_DEFAULTS.SEED_OFFSET

# PREFERRED -- inline literal
return base_seed + group_idx + 1
```

```python
# DISCOURAGED -- unnecessary constants for file paths
@dataclass(frozen=True)
class BootstrapDefaults:
    projects_dirname: str = "projects"
    required_child_dir: str = "closed_loop_rl"


BOOTSTRAP = BootstrapDefaults()

for parent in this_file.parents:
    if parent.name != BOOTSTRAP.projects_dirname:
        continue
    if not (parent / BOOTSTRAP.required_child_dir).is_dir():
        continue

# PREFERRED -- inline literals
for parent in this_file.parents:
    if parent.name != "projects":
        continue
    if not (parent / "closed_loop_rl").is_dir():
        continue
```

#### One way to do it

Each concept should have exactly one spelling. Do not add alternative
constructors, aliases, or "also accepts X" overloads.

```python
# DISCOURAGED -- multiple ways to configure the same thing
def connect(url=None, host=None, port=None, config=None):
    ...

# PREFERRED -- single explicit interface
def connect(url: str) -> Connection:
    ...
```

#### No dead-code compatibility. No backwards compatibility.

Do not keep parameters, branches, or adapters that exist only for
backward compatibility with removed code. Delete them.

We don't need backwards compatibility in APIs or method definitions. This is an
active research code base before release, not a public library. We are not
obligated to maintain any particular API or behavior. Instead, we optimize for
simple, clean code.

#### Solve the present problem, not a hypothetical future one

Do not add generality, configuration, or extension points for anticipated
needs. Write the simplest code that solves the current requirement.

```python
# DISCOURAGED -- speculative generality
class PipelineStep(ABC):
    @abstractmethod
    def run(self, ctx): ...

class LoadData(PipelineStep):
    def run(self, ctx):
        return load_csv(ctx.path)

# PREFERRED -- just do the thing
def load_data(path: str) -> DataFrame:
    return load_csv(path)
```

#### Tests prove behavior

Write tests only for meaningful behavior, functional and logical correctness and user-visible contracts, as well as regressions
that would be hard to notice from type checks or local reasoning alone. DO NOT WRITE
tests that only restate config defaults, mirror implementation details, or lock
down values that are already obvious at the call site.
For TDD: After implementation, clean up unnecessary tests according to the above principle.
Our goal is to have a small, but meaningful test suite that provides confidence in the code's behavior without being a maintenance burden.

#### Comments describe present code, not change history

Never add comments that reference what the old code did, why something was
"replaced", or what "used to" happen. Comments must make sense to a reader
who has never seen a previous version.

```python
# DISCOURAGED
x = compute_fast(data)  # replaced slow legacy path

# PREFERRED
x = compute_fast(data)
```

---
> Source: [NVlabs/alpagym](https://github.com/NVlabs/alpagym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
