## apex-research

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment

This project uses Docker containers for development. The primary setup command is:

```bash
./setup.sh          # installs Docker, builds container, installs CLI hooks
```

### Local Development Without Docker

For local development without Docker, use a Python virtual environment:

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # On macOS/Linux
# or
venv\Scripts\activate     # On Windows

# Install the package in development mode
pip install -e .

# Install test dependencies
pip install pytest pytest-asyncio
```

### Container Management

- **Open shell**: `docker exec -it $(docker ps -qf "name=aiagents") bash`
- **Alternative**: `devcontainer exec --workspace-folder . bash`
- **CPU development**: `make dev` or `make cpu`
- **GPU development**: `make dev-gpu` (requires CUDA host)
- **Rebuild container**: `make rebuild`
- **Clean containers**: `make clean`

### Common Development Commands

| Task                  | Command                                                            |
| --------------------- | ------------------------------------------------------------------ |
| **Lint/Format**       | `make lint` or `pre-commit run --all-files`                        |
| **Run tests**         | `pytest -q` or `make test` or `source venv/bin/activate && pytest` |
| **Launch JupyterLab** | `jupyter lab --ip 0.0.0.0 --port 8888 --no-browser`                |
| **Preview docs**      | `mkdocs serve -a 0.0.0.0:8000`                                     |
| **Run benchmarks**    | `make bench`                                                       |

### Running Experiments

To run experiments or custom training scripts:

```bash
# Activate virtual environment first
source venv/bin/activate

# Run your experiment script
python your_experiment.py
```

## Project Architecture

### Core Components

- **src/ai_agents_research/**: Core library with agent implementations, LLM utilities, and evaluation framework
- **benchmarks/**: Evaluation framework with official multi-agent benchmarks (MARBLE, TaskBench, AgentVerse)
- **third_party/**: Vendored implementations
- **tests/**: Unit and integration tests

### Benchmarking System

The benchmarking system uses a pluggable architecture in `benchmarks/core.py`:

- **DatasetLoader**: Loads and iterates over dataset samples
- **PromptBuilder**: Converts samples to chat messages
- **AnswerParser**: Extracts structured answers from raw LLM responses
- **Metric**: Computes evaluation scores (accuracy, F1, etc.)

Run benchmarks using the unified MARBLE runner:

```bash
# Quick MARBLE test (5 tasks, single vs APEX)
python run_marble.py --scenarios coding --max-tasks 5 --agent-types single apex_a2a

# Full MARBLE evaluation
python run_marble.py --scenarios coding reasoning --max-tasks 100 --agent-types single multi apex_a2a

# Resume interrupted benchmark
python run_marble.py --scenarios coding reasoning --max-tasks 100 --resume

# List available agent types
python run_marble.py --list-agents

# Compare agent types with baseline
python run_marble.py --compare apex_a2a --baseline single --scenarios coding

# Use specific LLM model
python run_marble.py --llm-model gpt-4 --llm-provider openai --scenarios reasoning
```

**Key Features of MARBLE Runner**:

- Checkpoint/resume capability for long runs
- Progress tracking with ETA
- Parallel benchmark execution
- Comprehensive performance metrics
- Agent comparison mode
- Multiple output formats (JSON, Markdown)

### Configuration

- **Python**: 3.11+ required
- **Package management**: Uses `pyproject.toml` with setuptools
- **Formatting**: Black (100 char line length) + Ruff linting
- **Testing**: pytest with asyncio support
- **Documentation**: MkDocs Material
- **Experiment tracking**: Weights & Biases integration
- **Version control**: DVC with Google Drive remote for large artifacts

### Key Integration Points

- **OpenAI API**: Uses OpenAI client with configurable base URLs (supports Ollama, LM-Studio)
- **Datasets**: Hugging Face datasets library for loading benchmark data
- **LLM cost tracking**: Built-in cost tracking decorator in `utils/wandb_logger.py`
- **Browser automation**: Playwright integration for web-based evaluations

## Data and Checkpoints

Large files are managed with DVC:

```bash
dvc add checkpoints/new_model.pt
git add checkpoints/new_model.pt.dvc
dvc push  # uploads to Google Drive
```

## Testing

Tests include both unit tests and notebook execution:

```bash
pytest -q                    # run all tests
pytest tests/               # unit tests only
pytest notebooks/          # execute notebooks
```

Use `--runslow` flag to include slow/long-running tests.

## Design

Refer to the APEX_Research_Design.txt document for context on the design of this repository (also available as APEX_Research_Design.pdf). Your tasks will be listed in IMPLEMENTATION.md, and you will use this file to look at tasks to complete as well as tasks to take on for the next section. It's very important you write incredibly clean code for this and make sure to run tests often to ensure that things are working properly. Write good tests as well.

### Design Documentation Guidelines

When creating design documents:

1. **Use Mermaid diagrams** to visualize architectures, workflows, and system interactions
2. Include diagrams for:
   - System architecture (before/after states)
   - Data flow between components
   - Sequence diagrams for complex interactions
   - State diagrams for agent lifecycles
3. Keep diagrams focused and readable - split complex systems into multiple diagrams
4. Use consistent styling and clear labels

.github/workflows/ci_cpu_gpu.yml is used for CI/CD. We need these to always pass.

You should always update the IMPLEMENTATION.md file with your changes and keep that as up to date as possible.

You must make sure to keep the requirements.txt updated. You must also ensure you create unit and integration tests for significant features.

You must do an alignment check after each prompting to ensure that the right problem is being solved.

Ensure you are always executing commands in the virtual environment.

## PR-Based Development Workflow

When working on tasks, I follow a pull request-based workflow:

### 1. Task Initiation

- Create a new feature branch: `git checkout -b username/task-description`
- Make all changes in this branch
- Run tests (`pytest -q`) and linting (`make lint`) before committing
- Update IMPLEMENTATION.md if needed

### 2. Commit and Push

- Commit with descriptive messages following conventional commits format
- Push to remote: `git push -u origin branch-name`
- Note: GitHub CLI (`gh`) needs to be installed and authenticated for PR creation

### 3. PR Creation

- Create PR using GitHub web interface or CLI (when authenticated)
- PR title should clearly describe the changes
- PR body should include:
  - Summary of changes
  - Test plan or verification steps
  - Any relevant context

### 4. Review Cycle

- Address all review comments in the same branch
- Make requested changes and push new commits
- Respond to comments explaining changes made
- Continue iterating until approval
- After addressing comments, post a comment on the PR summarizing what was done

### 5. Completion

- Once approved, the PR can be merged
- Return to main branch for next task: `git checkout main && git pull`

### Important Notes

- I will NOT merge PRs - that's the reviewer's responsibility
- I will NOT commit directly to main branch
- Each task gets its own PR for proper review
- All changes must pass tests and linting before pushing

## Documentation Guidelines

### Documentation Organization

- Prefer consolidating related documentation into existing files (like README.md)
- Avoid creating many separate documentation files
- When adding new sections to documentation:
  - Consider if it belongs in README.md first
  - Only create separate files for truly distinct, large topics
  - Keep quick reference and common workflows in README.md

### When Addressing PR Comments

1. Read and understand all review comments
2. Make the requested changes in the code
3. Test thoroughly to ensure nothing breaks
4. Commit with a descriptive message explaining what was addressed
5. Push the changes
6. **Important**: Comment on the PR using `gh pr comment` to acknowledge what was done

Example PR comment workflow:

```bash
# After addressing review comments and pushing changes
gh pr comment 17 --body "Addressed review comments:
- Consolidated TESTING_GUIDE.md and QUICK_START.md into README.md
- Deleted the separate markdown files as requested
- Organized content into logical sections
- All tests pass after changes"
```

## Code Consolidation Principles

When asked to consolidate code or reduce duplication:

### 1. Measure Success by Lines Removed

- **Success metric**: Net lines removed, not abstractions created
- If consolidation adds more code than it removes, reconsider the approach
- Track changes with `git diff --stat` to ensure you're actually simplifying

### 2. Only Abstract When There's Real Duplication

- **Rule of Three**: Only create abstractions when you have 3+ nearly identical implementations
- Don't create test fixtures for patterns that only appear 2-3 times
- Inline code is often clearer than complex abstractions

### 3. Complete the Migration

- When creating unified implementations, actually remove the old code
- Don't just create wrappers around existing code
- Ensure all imports are updated to use the new structure

### 4. Avoid Over-Engineering

- Test helpers should save significant lines (>50% reduction)
- Prefer simple, readable inline mocks over complex fixture systems
- Keep abstractions minimal and focused

### Example of What NOT to Do:

```python
# BAD: Created 800+ lines of test helpers to save 100 lines
# fixtures.py (336 lines)
# helpers/assertions.py (192 lines)
# helpers/builders.py (318 lines)
# Only used in 2-3 places each
```

### Example of What TO Do:

```python
# GOOD: Merged two 200-line services into one 400-line service
# Deleted the originals after updating imports
# Net reduction: ~200 lines with cleaner architecture
```

### Remember:

The goal is to make the codebase **cognitively simpler**, not to showcase abstraction skills. Every abstraction adds mental overhead - only add them when they provide significant value.

## Benchmark Integration Guidelines

**IMPORTANT: When implementing benchmarks, always use official datasets - never create synthetic or hallucinated data.**

### Current Benchmark Status

All major multi-agent benchmarks have been successfully integrated:

1. **MARBLE**: Full integration with official UIUC benchmark

   - All 6 scenarios supported
   - Real task evaluation using official metrics
   - Compatible with A2A architecture

2. **TaskBench**: Microsoft dataset via HuggingFace

   - 17.3k+ official tasks
   - Tool usage evaluation
   - API interaction testing

3. **AgentVerse**: OpenBMB framework integration
   - Multiple execution modes (CoT, Solo, Group)
   - Standard ML benchmarks adapted for agents
   - Performance comparison across modes

### Searching for Official Benchmarks

When asked to implement or integrate a benchmark:

1. **Always search online first** - Use WebSearch to find the official benchmark repository or dataset
2. **Verify authenticity** - Look for official GitHub repos, papers, or dataset releases
3. **Use official implementations** - Never create synthetic "sample" data or hallucinate benchmark tasks
4. **Document sources** - Always include links to official repositories in code comments

### Official Benchmark Links

- **MARBLE (Multi-Agent Reasoning Benchmark)**: https://github.com/ulab-uiuc/MARBLE

  - 6 official scenarios: coding, database, reasoning, research, werewolf, world
  - Installation instructions:

    ```bash
    # Option 1: Install via pip (when available)
    pip install marble-bench

    # Option 2: Local installation
    git clone https://github.com/ulab-uiuc/MARBLE.git
    cd MARBLE
    poetry install  # or pip install -e .

    # Option 3: Use with path
    # When running benchmarks, specify --marble-path /path/to/MARBLE
    ```

- **TaskBench**: https://huggingface.co/datasets/microsoft/Taskbench
  - Official Microsoft dataset with 17.3k+ samples
  - 3 domains: dailylifeapis, huggingface, multimedia
  - Already integrated via HuggingFace datasets library
- **AgentVerse**: https://github.com/OpenBMB/AgentVerse
  - Multi-agent framework with official benchmarks
  - Includes MGSM, HumanEval, CommonGen, Logic Grid
  - Supports CoT, Solo, and Group execution modes

### Year Context

**Current Year: 2025 (July)** - When searching for benchmarks or documentation, use recent years (2024-2025) in search queries to find the latest versions. The project has been under active development throughout 2025.

## LLM Integration Architecture

**IMPORTANT: If LLM tasks seem unclear, check IMPLEMENTATION.md TODO section for current status and details.**

### Current LLM Architecture (July 2025)

The codebase is being refactored to use a unified LLM client architecture:

1. **UnifiedLLMClient**: Single interface for all LLM providers (Ollama, OpenAI, Anthropic, Mock)

   - Location: `src/ai_agents_research/llm/unified_client.py`
   - Uses LiteLLM for provider abstraction and cost tracking
   - Default: Ollama with gemma3:latest for local development

2. **Current LLM Patterns Being Replaced**:
   - Direct OpenAI client usage
   - Legacy OpenAI async patterns
   - Langchain Ollama integration
   - Generic client patterns
   - Various mock implementations
3. **Key Features**:
   - Automatic cost tracking via LiteLLM (no custom implementation needed)
   - No artificial token limits - models generate what they need
   - Easy provider switching via configuration
   - Consistent error handling across providers

### Task Tracking

All LLM integration tasks are tracked in `IMPLEMENTATION.md` under the "Current TODO Status" section. This includes:

- Unified LLM client implementation progress
- Testing and validation status

If you encounter issues or need to understand the current state of LLM integration, always refer to IMPLEMENTATION.md first.

## Task Completion and Cleanup

When completing any significant task, follow this cleanup checklist:

### 1. Run Tests

```bash
source venv/bin/activate
PYTHONPATH=/path/to/project/src:$PYTHONPATH pytest tests/ -q
```

- Ensure at least 95% of tests pass
- Fix any broken tests if they were caused by your changes
- Document any pre-existing failures

### 2. Remove Temporary Files

After completing a task, clean up:

- Remove any temporary scripts created for testing or fixes
- Delete planning documents (*PLAN.md, *STATUS.md) after implementation
- Remove empty directories and placeholder files
- Clean up any debug logs or output files

### 3. Check for Obsolete Files

- Look for duplicate implementations that can be consolidated
- Remove commented-out code blocks
- Delete unused imports and variables
- Remove files made obsolete by refactoring

### 4. Final Verification

Before marking a task complete:

- Verify the main functionality works as expected
- Ensure no unnecessary files were left behind
- Confirm documentation is updated if needed
- Run a final test to ensure nothing broke during cleanup

### Example Cleanup Commands

```bash
# Remove common temporary files
rm -f fix_*.py test_*.py mock_*.py *_temp.* *_old.*

# Find and remove empty directories
find . -type d -empty -delete

# Find potentially obsolete markdown files
find . -name "*PLAN*.md" -o -name "*STATUS*.md" -o -name "*SUMMARY*.md"

# Check for unused imports (requires flake8)
flake8 src/ benchmarks/ --select=F401,F841
```

### Important: Always clean up after yourself

Every task should leave the codebase in a better state than you found it. This means:

- Removing temporary files you created
- Consolidating duplicate code when found
- Updating tests to match new APIs
- Ensuring the codebase remains clean and maintainable

# important-instruction-reminders

Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (\*.md) or README files. Only create documentation files if explicitly requested by the User.
ALWAYS refactor and update existing documents or code rather than creating brand new ones when possible.

## CRITICAL BENCHMARK REQUIREMENTS

We MUST make official benchmarks work. We CANNOT skimp on benchmarks or create our own hallucinated/synthetic benchmarks. This is a hard line. We must find models that can actually complete the official benchmark tasks, not create easier alternatives.

## Benchmark Evaluation Rules

**CRITICAL**: When working with benchmarks (MARBLE, SWE-bench, etc.):

1. NEVER create custom evaluators or scoring functions
2. ALWAYS use the official evaluation from the benchmark repository
3. ALWAYS report results using the official metrics only
4. If you create any test evaluators, you MUST delete them before reporting results
5. NEVER modify evaluation logic to make results look better
6. If the official evaluator requires external APIs (e.g., OpenAI), inform the user what's needed

# Subagent usage

Use subagents liberally and as much as possible to save context window. Make sure that they are delegated tasks that are of reasonable scope. Always prefer giving a plan to me first before continuing to implementation.

# Plan creation & Implementation

Prefer giving solutions that solve the root cause of problems rather than patches or mitigations that will be annoying to fix later. We want robust code that works well. Prefer updating the existing code rather than creating brand new files when possible.

Delete code that is unnecessary or dead to keep the codebase clean as much as possible

# Always prefer using subagents

# Always prefer editing existing files over creating new ones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SujeethJinesh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SujeethJinesh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
