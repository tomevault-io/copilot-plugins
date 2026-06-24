## agents-cli-shadow-ci-win-w73hxa55

> Manages GitHub Actions WIF, service accounts, APIs, storage, and IAM for the CI/CD infrastructure using `agents-cli-test-*` projects.

# agents-cli — AI Coding Agent Guide

> **Scope**: This document is for AI coding agents contributing to the **Agents CLI in Agent Platform** repository itself (the CLI tool that scaffolds, develops, evaluates, and deploys ADK agents). For guidance on working with **generated projects**, see the bundled skills under `skills/`.

---

## Core Principles for AI Agents

1.  **Preserve and Isolate:** Modify *only* the code segments directly related to the user's request — preserve all surrounding code, comments, and formatting.
2.  **Follow Conventions:** Analyze existing patterns before writing new code; replicate naming, templating logic, and directory structure conventions.
3.  **Template-First Mindset:** The CLI should remain lean with good defaults. Most features belong in templates, not CLI code.
4.  **Search Comprehensively:** A single change often requires updates in multiple places across templates, CLI code, and CI/CD configuration.

---

## Project Architecture Overview

### Package Structure

```
src/google/agents/cli/
├── __init__.py              # __version__
├── main.py                  # Root Click group, registers all commands
├── _output.py               # emit(), ExitCode enum (JSON structured output)
├── _project.py              # read_project_config() — reads configuration manifest from agents-cli-manifest.yaml
├── _runner.py               # run() — unified subprocess helper
├── _trust.py                # require_confirmation() decorator
├── auth.py                  # Authentication helpers
│
├── setup/                   # Skills installation + auth commands
│   ├── cmd_setup.py         # agents-cli setup — install bundled skills via npx
│   ├── cmd_update.py        # agents-cli update — update skills via npx
│   └── cmd_auth.py          # agents-cli login/status
│
├── scaffold/                # Project scaffolding (vendored from Agent Starter Pack)
│   ├── commands/
│   │   ├── create.py        # agents-cli scaffold create (alias: agents-cli create)
│   │   ├── enhance.py       # agents-cli scaffold enhance
│   │   └── upgrade.py       # agents-cli scaffold upgrade
│   ├── utils/               # Template processing, helpers, lock generation
│   ├── agents/              # Agent templates (adk, adk_a2a, agentic_rag)
│   ├── base_templates/      # Base project templates (python/)
│   ├── deployment_targets/  # Deployment overrides (agent_runtime, cloud_run, gke)
│   └── resources/           # Locks, docs, IDX configs
│
├── dev/                     # Development commands (thin wrappers)
│   ├── cmd_playground.py    # agents-cli playground — start local playground
│   ├── cmd_lint.py          # agents-cli lint — ruff check + format
│   └── cmd_install.py       # agents-cli install — uv sync
│
├── run/                     # Run command
│   └── cmd_run.py           # agents-cli run — run agent with a single prompt
│
├── data/                    # Data/RAG commands
│   ├── _helpers.py             # Shared helpers (resolve_project_id, require_rag_project, …)
│   └── cmd_data_ingestion.py   # agents-cli data-ingestion
│
├── eval/                    # Evaluation commands
│   ├── cmd_analyze.py       # agents-cli eval analyze
│   ├── cmd_compare.py       # agents-cli eval compare
│   ├── cmd_dataset.py       # agents-cli eval dataset
│   ├── cmd_generate.py      # agents-cli eval generate
│   ├── cmd_grade.py         # agents-cli eval grade
│   ├── cmd_metric.py        # agents-cli eval metric
│   └── cmd_optimize.py      # agents-cli eval optimize
│
├── deploy/                  # Deployment commands
│   ├── cmd_deploy.py        # agents-cli deploy — dispatches by deployment_target
│   └── agent_runtime.py      # Agent Runtime deployment logic
│
├── infra/                   # Infrastructure commands
│   ├── cmd_infra.py         # agents-cli infra single-project — single-project terraform
│   ├── cmd_cicd.py          # agents-cli infra cicd — CI/CD pipeline + staging/prod infra
│   └── cmd_datastore.py     # agents-cli infra datastore — RAG datastore provisioning
│
├── publish/                 # Publish commands
│   └── cmd_publish.py       # agents-cli publish gemini-enterprise
│
├── info/                    # Info command
│   └── cmd_info.py          # agents-cli info — show project config and CLI version
│
└── skills/                  # SKILLS_DATA_DIR path resolution
    └── __init__.py          # Resolves to root skills/ (source) or bundled data/ (wheel)

skills/                      # Bundled IDE skills at repo root
├── google-agents-cli-workflow/
├── google-agents-cli-adk-code/
├── google-agents-cli-scaffold/
├── google-agents-cli-eval/
├── google-agents-cli-deploy/
├── google-agents-cli-publish/
└── google-agents-cli-observability/
```

### Bundled Skills

Skills are installed to coding agents via `agents-cli setup` and provide context-specific guidance for working with generated agent projects.

- **google-agents-cli-workflow** — Always-active development lifecycle guide. Covers the spec-driven workflow (understand, build, evaluate, deploy), code preservation rules, model selection, and troubleshooting.
- **google-agents-cli-adk-code** — ADK Python API quick reference. Agent types, tool definitions, orchestration patterns, callbacks, state management, and an index of all ADK documentation pages.
- **google-agents-cli-scaffold** — Project creation and enhancement guide. Covers `scaffold create`, `scaffold enhance`, `scaffold upgrade` commands, template options, deployment targets, and the prototype-first workflow.
- **google-agents-cli-eval** — Agent evaluation methodology. Eval metrics, dataset schema, the eval-fix loop, LLM-as-judge scoring, and common failure causes.
- **google-agents-cli-deploy** — Deployment to Google Cloud. Agent Runtime vs Cloud Run vs GKE decision matrix, CI/CD pipelines, secrets, service accounts, and rollback.
- **google-agents-cli-publish** — Platform registration. Gemini Enterprise registration workflows, ADK vs A2A modes, CI/CD examples, and troubleshooting.
- **google-agents-cli-observability** — Production observability setup. Cloud Trace, prompt-response logging, BigQuery Agent Analytics, third-party integrations (AgentOps, Phoenix, MLflow, etc.), and troubleshooting.

---

### 3-Layer Template System

Template processing follows this hierarchy (later layers override earlier ones):

| Layer | Directory | Purpose |
|-------|-----------|---------|
| 1. Base | `scaffold/base_templates/python/` | Core Jinja scaffolding |
| 2. Deployment | `scaffold/deployment_targets/` | Environment overrides (cloud_run, gke, agent_runtime) |
| 3. Agent | `scaffold/agents/*/` | Agent-specific logic and configurations |

### Available Agents and Deployment Targets

| Agent | Description | Supported Targets |
|-------|-------------|-------------------|
| `adk` | Base ADK agent | agent_runtime, cloud_run, gke |
| `adk_a2a` | A2A-enabled ADK agent | agent_runtime, cloud_run, gke |
| `agentic_rag` | RAG agent with datastore | agent_runtime, cloud_run, gke |

### When to Modify What

| Change Type | Where to Modify | Also Check |
|-------------|-----------------|------------|
| Affects ALL generated projects | `scaffold/base_templates/python/` | Deployment targets for conflicts |
| Deployment-specific logic | `scaffold/deployment_targets/<target>/` | Base templates for shared code |
| Agent-specific feature | `scaffold/agents/<agent>/` | Other agents for consistency |
| New CLI command/flag | CLI modules under `src/google/agents/cli/` | Existing commands for patterns |
| CI/CD changes | `.github/` | May also need to adjust integration or e2e tests |

---

## Architecture

- **Lazy command loading**: Every CLI command is registered via `LazyGroup.add_lazy_command(name, "module:obj", short_help)` in `main.py`. Command modules are imported only when invoked. New commands must follow this pattern — `tests/unittests/cli/test_click.py` enforces it (no eager `add_command` at root or in any lazy group, and `short_help` strings must match each command's docstring summary).
- **Thin wrappers**: Most commands (`playground`, `run`, `lint`, `install`) are thin wrappers that call underlying tools (`adk`, `ruff`, `uv`) via `run()`, which prints the command before executing.
- **Dependency sync**: `lint` and `eval` run `uv sync` with appropriate extras before executing (matching the generated Makefile behavior).
- **Scaffolding**: The `scaffold create`, `scaffold enhance`, `scaffold upgrade`, and `infra cicd` commands use vendored scaffolding code (originally from Agent Starter Pack). All imports rewritten to `google.agents.cli.scaffold.*`. `agents-cli create` is a top-level alias for `agents-cli scaffold create`.
- **Project config**: Commands read configuration from the user's `agents-cli-manifest.yaml` file. Creation parameters like `deployment_target` are defined under the `create_params:` key at the manifest level.
- **Skills setup**: Uses `npx skills` CLI to install bundled skills. Always runs in programmatic mode (`-y --all`). Skills source is the local bundled directory at `SKILLS_DATA_DIR`.
- **Post-scaffold injection**: After `agents-cli scaffold create`, `inject_agents_cli()` adds `agents-cli` as a dev dependency (auto-detecting local vs published install) and writes the agent guidance file.

---

## CLI Commands

```bash
agents-cli setup              # Install skills to coding agents
agents-cli login              # Authenticate with Google Cloud or AI Studio
agents-cli login --status     # Show authentication status
agents-cli create my-agent    # Create new agent project (alias for scaffold create)
agents-cli scaffold create    # Create new agent project
agents-cli scaffold enhance   # Add deployment/CI-CD to project
agents-cli scaffold upgrade   # Upgrade project to newer version
agents-cli playground         # Start local playground (adk web)
agents-cli run "prompt"       # Run agent with a single prompt
agents-cli install            # Install project dependencies
agents-cli lint               # Run linting (ruff)
agents-cli info               # Show project config and CLI version
agents-cli eval generate      # Run agent inference over eval cases
agents-cli eval grade         # Grade generated traces
agents-cli eval dataset synthesize  # Synthesize multi-turn eval scenarios for your agent
agents-cli eval compare       # Compare two eval result files
agents-cli eval analyze       # Cluster failure modes from grade results
agents-cli eval metric list   # List available built-in metrics
agents-cli eval optimize      # Auto-tune agent prompts using eval data
agents-cli deploy             # Deploy agent
agents-cli infra single-project  # Provision single-project infrastructure
agents-cli infra cicd         # Set up CI/CD pipeline
agents-cli infra datastore    # Provision datastore infra for RAG
agents-cli data-ingestion     # Run data ingestion for RAG
agents-cli publish gemini-enterprise  # Register with Gemini Enterprise
agents-cli update             # Force reinstall skills to all IDEs

```

---

## Building

```bash
uv build --wheel
```

---

## Template Development Workflow

Template changes require a specific workflow because you're modifying Jinja templates, not regular source files.

### Step-by-Step Process

#### 1. Generate a Test Instance

```bash
uv run agents-cli scaffold create mytest -p -s -y -d cloud_run --output-dir target
```

Flags: `-p` (prototype), `-s` (skip checks), `-y` (auto-approve), `-d` (deployment target), `--output-dir target` (gitignored output).

#### 2. Initialize Git in the Generated Project

```bash
cd target/mytest && git init && git add . && git commit -m "Initial"
```

#### 3. Develop with Tight Feedback Loops

Make changes directly in `target/mytest/`, test immediately, then use `git diff` to see exactly what changed.

#### 4. Backport Changes to Jinja Templates

Find the source template and apply your changes with appropriate Jinja conditionals.

#### 5. Validate Across Combinations

```bash
# Test your target combination
_TEST_AGENT_COMBINATION="adk,cloud_run,--session-type,in_memory" make lint-templated-agents

# Test alternate agent with same deployment
_TEST_AGENT_COMBINATION="adk_a2a,cloud_run" make lint-templated-agents

# Test same agent with alternate deployment
_TEST_AGENT_COMBINATION="adk,agent_runtime" make lint-templated-agents
```

---

## Jinja Templating Rules

### Block Balancing (Critical)

Every opening Jinja block must have a corresponding closing block: `{% if %}` → `{% endif %}`, `{% for %}` → `{% endfor %}`.

### Whitespace Control

Use hyphens to control newlines: `{%-` removes whitespace before, `-%}` removes whitespace after, `{%- -%}` removes both sides.

### Critical Whitespace Patterns

**Conditional imports with blank line separation:**
```jinja
from opentelemetry.sdk.trace import TracerProvider, export
{% if cookiecutter.session_type == "agent_platform_sessions" -%}
from vertexai import agent_engines
{% endif %}

from {{cookiecutter.agent_directory}}.app_utils.gcs import create_bucket_if_not_exists
```

**File end newlines:** Ruff requires exactly ONE newline at end of every file. Watch `{%- endif %}` blocks carefully.

---

## Testing Strategy

### Testing Coverage Matrix

| Dimension | Options |
|-----------|---------|
| Agents | adk, adk_a2a, agentic_rag |
| Deployments | cloud_run, gke, agent_runtime |
| Session types | in_memory, cloud_sql, agent_platform_sessions |

### Makefile Targets

```bash
make test                    # Run all tests (uv run pytest tests)
make test-templated-agents   # Run template pattern tests
make test-e2e                # Run E2E deployment tests (requires .env)
make lint                    # Ruff check/format + ty type check
make lint-templated-agents   # Lint generated templates
make generate-lock           # Regenerate uv.lock files for templates
make install                 # Install all dev dependencies (frozen)
make clean                   # Remove target/ build artifacts
make simulate                # Run agent simulator on Cloud Build (default: support_bot)
make test-windows            # Trigger Shadow CI run for Windows compatibility
```

#### Windows Testing (Shadow CI)

Since local development is primarily on Linux, Windows compatibility is verified via **Shadow CI**. This system clones your current local changes into a temporary remote GitHub repository and triggers a GitHub Actions run on a Windows runner.

**Usage:**
```bash
uv run tools/ci/run_windows_shadow_ci.py
```
Or via the make target:
```bash
make test-windows
```

**Requirements:**
- GitHub CLI (`gh`) must be installed and authenticated.
- You must have permissions to create repositories in the configured org (defaulting to your user).
- Changes must be committed or staged for the Shadow CI to pick them up (it uses `git clone` or a temporary branch).

#### Agent Simulator

Run e2e agent-building scenarios on Cloud Build:

```bash
make simulate                          # support_bot scenario (default)
make simulate SCENARIO=smoke_test      # quick scaffold + eval only
make simulate SCENARIO=rag_agent       # RAG agent with Vector Search
make simulate PROJECT_ID=my-project    # run on a different GCP project
```

Scenarios live in `tools/simulator/scenarios/`. Results are uploaded to `gs://acli-test-cicd-logs-data/simulator/`. See `tools/simulator/README.md` for details.

`tools/simulator/cloudbuild/` contains relevant cloud build files, and `tools/simulator/containers/` contains Docker images.

### Common Test Combinations

```bash
# Cloud Run combinations
_TEST_AGENT_COMBINATION="adk,cloud_run,--session-type,in_memory" make lint-templated-agents

# Agent Runtime combinations
_TEST_AGENT_COMBINATION="adk,agent_runtime" make lint-templated-agents
_TEST_AGENT_COMBINATION="adk_a2a,agent_runtime" make lint-templated-agents

# GKE combinations
_TEST_AGENT_COMBINATION="adk,gke,--session-type,in_memory" make lint-templated-agents

# RAG agent
_TEST_AGENT_COMBINATION="agentic_rag,agent_runtime,--datastore,agent_platform_search" make lint-templated-agents
```

### Minimum Coverage Before PR

- [ ] Your target combination
- [ ] One alternate agent with same deployment target
- [ ] One alternate deployment target with same agent

### Testing Conventions

- **Test the public API, not private functions.** Avoid writing dedicated test classes for `_`-prefixed helpers — they couple tests to implementation details and break on refactors. Instead, test the public command/function and let it exercise internals. If a private function is complex enough to need its own test grid, consider promoting it to a public function in its own module.
- **Mock at boundaries, not internals.** Avoid `@mock.patch` on private functions from the same module — it makes tests brittle because any refactor (rename, inline, extract) breaks them. Mock at external boundaries instead: `subprocess.run`, network calls, filesystem.
- **Use pyfakefs for filesystem tests.** Prefer the `fs` fixture (from pyfakefs) over `tmp_path` + mocks. It provides a fake in-memory filesystem that isolates tests without touching disk:
  ```python
  def test_reads_config(self, fs):
      fs.create_file("/project/pyproject.toml", contents='[project]\nname = "my-agent"\n')
      cfg = read_project_config("/project")
      assert cfg.project_name == "my-agent"
  ```
- **Keep test files aligned with source.** Tests for `src/.../publish/cmd_publish.py` belong in `tests/.../test_publish.py`, not scattered across unrelated directories.
- **Don't over-test.** Prefer a few focused tests on the public function over many small test classes for each private helper. Redundant tests add maintenance burden without improving coverage.

---

## CI/CD Integration

### GitHub Actions (`.github/workflows/`)

All CI/CD pipelines run on GitHub Actions using self-hosted runners with Workload Identity Federation for GCP auth.

### E2E Test Bot Account

The E2E deployment tests use a GitHub bot account (`agents-cli-dev-bot`). The bot password is stored in Secret Manager: `projects/569187589892/secrets/GH_BOT_PASSWORD`. This is needed to set up the GitHub account and access test repositories created during E2E runs.

### Terraform (`.github/terraform/`)

Manages GitHub Actions WIF, service accounts, APIs, storage, and IAM for the CI/CD infrastructure using `agents-cli-test-*` projects.

> **Note on Terraform changes:** Validate locally with `cd .github/terraform && terraform init -backend=false && terraform validate`, and use `terraform plan` to preview changes against the real state.

---

## Import Patterns

All internal imports use the `google.agents.cli` namespace:

```python
from google.agents.cli.scaffold.utils.template import process_template
from google.agents.cli._runner import run
from google.agents.cli._output import emit, ExitCode
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/google/agents/cli/scaffold/commands/create.py` | Main create command, CLI flags |
| `src/google/agents/cli/scaffold/utils/template.py` | Template processing, `process_template()` |
| `src/google/agents/cli/scaffold/base_templates/python/pyproject.toml` | Generated project metadata |
| `src/google/agents/cli/scaffold/utils/generate_locks.py` | Lock file generation |
| `src/google/agents/cli/scaffold/utils/lock_utils.py` | Lock file path utilities |

---

## Debugging Linting Failures

1. **Identify the exact error** from the diff output
2. **Find the generated file** in `target/`
3. **Trace back to template**: search `src/google/agents/cli/scaffold/` for the filename
4. **Check BOTH branches** of conditionals — test with condition true AND false

### Common Linting Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Missing blank line between imports | Conditional import without proper spacing | Add blank line inside `{% if %}` block |
| Extra blank line | Jinja block creating unwanted newline | Use `{%- endif -%}` |
| Missing newline at end of file | Template ends without final newline | Ensure exactly one blank line at end |
| Line too long | Import statement exceeds limit | Split into multi-line with parentheses |

---

## Common Pitfalls

- **Namespace packages**: `src/google/__init__.py` and `src/google/agents/__init__.py` must NOT exist
- **Config key**: Generated projects use `agents-cli-manifest.yaml` in their project root directory, with creation parameters defined under the `create_params` manifest key
- **Entry point**: `agents-cli` binary defined in `pyproject.toml` → `google.agents.cli.main:main`
- **Missing conditionals**: Wrap agent-specific code in proper `{% if %}` blocks
- **Dependency conflicts**: Some agents lack certain extras

---

## Code Conventions

- **Single source of truth for constants.** Extract magic strings and repeated values into module-level constants. Import the constant rather than duplicating the string. Example: `METADATA_FILE` is defined once in `deploy/_operation.py` and imported wherever needed.
- **Don't silently swallow exceptions.** Avoid bare `except Exception: pass`. At minimum, log the error with `logging.warning()`. If a silent fallback is intentional (e.g., graceful degradation for optional features), add a comment explaining the rationale.
- **Use `logging.warning()` for all warnings.** Don't use `click.echo(..., err=True)` for warnings — we are converging on `logging` everywhere (see b/499673325). Use `click.echo()` only for normal user-facing CLI output (stdout).
- **Provide actionable error messages.** Tell the user what went wrong AND how to fix it. Example: *"data_ingestion folder is missing — rerun `agents-cli scaffold enhance` to restore it"* is better than *"folder not found"*.
- **Comment non-obvious behavior.** If code does something surprising (e.g., overriding a URL because of kubectl port-forward semantics), add a brief comment explaining why. Don't comment obvious code.
- **Python 3.11 compatibility.** The project requires `>=3.11`. Avoid 3.12+ APIs.
- **CLI flag naming.** Follow gcloud conventions. Use `--yes` / `--auto-approve` (via the `@require_confirmation()` decorator in `_trust.py`) for non-interactive mode. Use `--force` only for overwrite/destructive semantics, not as a synonym for auto-approve.
- **Config key placement.** New project-level configuration parameters go under the `create_params:` key in `agents-cli-manifest.yaml`, not at the top level of the manifest layout.

---

## Pull Request Best Practices

Use `ggh` (not `gh`) for all GitHub CLI operations (creating PRs, listing checks, etc.). It has the same interface as `gh` but is configured for the corp GitHub instance.

### Commit Message Format

```
<type>: <concise summary in imperative mood>
```

**Types**: `fix`, `feat`, `refactor`, `docs`, `test`, `chore`

### PR Labels

When opening a PR that changes files under `skills/`, add the `skills` label: `ggh pr create --label skills ...`

### PR Structure

```markdown
## Summary
- Key change 1
- Key change 2

## Changes
- What was modified and why
```

### PR Review Guidelines

- **Keep PRs small and focused.** Separate functional changes (renames, refactors, feature additions) into distinct PRs. Don't combine unrelated changes — it makes review harder and increases risk.
- **Check ripple effects.** When renaming a flag, constant, or file, search the entire codebase — templates, tests, docs, and CI configs — for all references. A partial rename is worse than no rename.

---
> Source: [agents-cli-dev-bot/agents-cli-shadow-ci-win-w73hxa55](https://github.com/agents-cli-dev-bot/agents-cli-shadow-ci-win-w73hxa55) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
