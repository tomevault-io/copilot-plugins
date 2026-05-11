## kuberoku

> Run `pytest tests/ -v && mypy src/kuberoku/ --strict && ruff check src/ tests/ && ruff format --check src/ tests/` to verify any file edits before prompting for input.

# Agent Instructions for kuberoku

Run `pytest tests/ -v && mypy src/kuberoku/ --strict && ruff check src/ tests/ && ruff format --check src/ tests/` to verify any file edits before prompting for input.

## What this is

Kuberoku is a Python CLI + SDK that gives developers Heroku-like DX on vanilla Kubernetes. Zero server-side components — CLI/SDK only, K8s is the database. The complete specification is in `docs/NORTHSTAR.txt`.

## Architecture (strict three-layer)

```
CLI (Click)  -->  SDK (business logic)  -->  K8s (protocol + client)
```

- **CLI never touches K8s directly.** Calls one SDK method, formats the result.
- **SDK never prints.** Returns frozen dataclasses, raises typed exceptions, progress via `on_step` callback.
- **K8s client returns `dict[str, Any]`.** No K8s model objects leak upward. `typing.Protocol` defines the interface.
- **KuberokuFactory** is the DI container. Tests inject `FakeK8sClient`.

## Required before each commit

```bash
pytest tests/ -v                    # tests pass
mypy src/kuberoku/ --strict         # types check
ruff check src/ tests/              # lint clean
ruff format --check src/ tests/     # format clean
```

## Source layout

```
src/kuberoku/
    branding.py          # TOOL_NAME + all derived constants (single source)
    models.py            # Frozen dataclasses (App, Release, Dyno, etc.)
    exceptions.py        # KuberokuError hierarchy
    factory.py           # KuberokuFactory (DI container)
    client.py            # Public facade: from kuberoku import Kuberoku
    k8s/
        protocols.py     # K8sClientProtocol (typing.Protocol)
        client.py        # Real K8s client (wraps kubernetes library)
        labels.py        # Label builders: for_app(), for_process(), selector()
        resources.py     # Resource dict builders: build_deployment(), safe_name()
    sdk/                 # One service per command group (apps.py, config.py, deploy.py, ...)
    cli/                 # One file per command group, mirrors sdk/ 1:1
        main.py          # ColonCommandGroup (colon commands), root group, entry point
tests/
    backends/fake.py     # FakeK8sClient (in-memory, implements K8sClientProtocol)
    {feature}/           # tests/{apps,config,deploy,...}/test_{command}.py
    infrastructure/      # tests for labels, resources, models, branding, factory
    contract/            # FakeK8sClient matches real K8s behavior
```

## Key rules

- **Branding**: All identity constants derive from `branding.py`. No hardcoded "kuberoku" strings elsewhere.
- **Resource names**: `{PREFIX}-{type}-{app}[-{suffix}]`. Use `safe_name()` for 63-char K8s limit.
- **Labels**: `{DOMAIN}/managed-by`, `{DOMAIN}/app`, `{DOMAIN}/process-type`. See `k8s/labels.py`.
- **Optimistic concurrency**: All ConfigMap/Secret updates use `resourceVersion` + retry on 409.
- **Colon commands**: `apps:create` and `apps create` both work via `ColonCommandGroup.resolve_command()`.
- **LazyGroup**: Don't import sdk/ or k8s/ at CLI module level. Keeps `--help` under 100ms.
- **Cross-platform**: pathlib everywhere, no `shell=True`. Windows + Linux + macOS.

## Development workflow (TDD per command)

1. Write `tests/{feature}/test_{command}.py` first — SDK + CLI tests in same file
2. Implement SDK method in `src/kuberoku/sdk/`
3. Wire CLI command in `src/kuberoku/cli/`
4. Verify: tests, mypy, ruff all pass

## NORTHSTAR.txt section index

Read the relevant section when you need deeper context:

- Section 1: Vision & positioning
- Section 2: Branding & naming
- Section 3: End-to-end user experience
- Section 4: Command reference
- Section 5: RBAC & permissions
- Section 6: Colon command implementation
- Section 7: SDK API design
- Section 8: Non-HTTP services & multi-port
- Section 9: Architecture
- Section 10: Data models
- Section 11: Exception hierarchy
- Section 12: Testing strategy (backends, fixtures, contract tests, TDD workflow)
- Section 13: Dependencies
- Section 14: Distribution
- Section 15: Plugin system
- Section 16: Implementation phases (0-14)
- Section 17: Complete command summary
- Section 18: App name resolution
- Section 19: Open questions & future work

## Dependencies

- Runtime: click >=8.1, kubernetes >=35, pyyaml >=6
- Dev: pytest, pytest-cov, pytest-mock, mypy (--strict), ruff, vulture, pip-audit, types-PyYAML
- Build: hatchling. Entry point: `kuberoku = "kuberoku.cli.main:run"`

---
> Source: [amanjain/kuberoku](https://github.com/amanjain/kuberoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
