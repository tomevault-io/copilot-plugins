## wenqiao

> - Do what has been asked; nothing more, nothing less

# Claude Code Configuration - Claude Flow V3

## Behavioral Rules (Always Enforced)

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested
- NEVER save working files, text/mds, or tests to the root folder
- Never continuously check status after spawning a swarm — wait for results
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- ALWAYS comment in bot Chinese and English

## File Organization

- NEVER save to root folder — use the directories below
- Use `/src` for source code files
- Use `/tests` for test files
- Use `/docs` for documentation and markdown files
- Use `/config` for configuration files
- Use `/scripts` for utility scripts
- Use `/examples` for example code

## Project Architecture

- Follow Domain-Driven Design with bounded contexts
- Keep files under 500 lines
- Use typed interfaces for all public APIs
- Prefer TDD London School (mock-first) for new code
- Use event sourcing for state changes
- Ensure input validation at system boundaries

## Python Coding Standards / Python 代码规范

### Type Annotations / 类型注解

- **ALL** functions and methods MUST have complete type annotations
- Use `from __future__ import annotations` for forward references
- Use generics properly: `list[str]`, `dict[str, int]`, `Callable[[int], str]`
- Use `| None` instead of `Optional`
- Use `... | ...` instead of `Union[...]`
- Complex return types should use type aliases or NamedTuple

```python
# ✅ Good
from __future__ import annotations

def process_data(items: list[dict[str, str]]) -> dict[str, int] | None:
    ...

# ❌ Bad
def process_data(items):
    ...
```

### Comments / 注释规范

- **ALL** comments MUST be bilingual: English + Chinese
- Format: `English description (中文说明)`
- Docstrings follow Google style with bilingual support

```python
# ✅ Good
# Calculate the average value (计算平均值)
def calculate_average(values: list[float]) -> float:
    """Calculate arithmetic mean of values.
    
    计算数值的算术平均值。
    
    Args:
        values: List of numeric values (数值列表)
        
    Returns:
        The arithmetic mean (算术平均数)
        
    Raises:
        ValueError: If values is empty (当列表为空时)
    """
    if not values:
        raise ValueError("Empty list (列表不能为空)")
    return sum(values) / len(values)

# ❌ Bad
# Calculate average
def calculate_average(values):
    """Calculate mean."""
    return sum(values) / len(values)
```

### Docstring Standards / 文档字符串规范

- Use triple double quotes `"""`
- Include: Description (EN+CN), Args, Returns, Raises
- Type info in docstrings is optional if type annotations are complete

```python
class DataProcessor:
    """Process and transform data structures.
    
    处理和转换数据结构。
    
    Attributes:
        config: Configuration dictionary (配置字典)
        cache: Internal cache for results (结果缓存)
    """
    
    def __init__(self, config: dict[str, str]) -> None:
        """Initialize processor with configuration.
        
        使用配置初始化处理器。
        
        Args:
            config: Processing configuration (处理配置)
        """
        self.config = config
        self.cache: dict[str, object] = {}  # Internal cache (内部缓存)
```

### Naming Conventions / 命名规范

| Type | Convention | Example |
|------|------------|---------|
| Module | `snake_case` | `comment_processor.py` |
| Class | `PascalCase` | `LaTeXRenderer` |
| Function | `snake_case` | `process_comments()` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_LINE_LENGTH` |
| Private | `_leading_underscore` | `_internal_helper()` |
| Type Variable | `PascalCase` or `T`, `K`, `V` | `NodeT` |

### Best Practices / 最佳实践

1. **Explicit over Implicit / 显式优于隐式**
   - Always use explicit imports (使用显式导入)
   - Avoid `from module import *` (避免星号导入)

2. **Fail Fast / 快速失败**
   - Validate inputs at function entry (在函数入口验证输入)
   - Raise specific exceptions with clear messages (抛出具体异常)

3. **Immutability / 不可变性**
   - Use `dataclasses` with `frozen=True` for value objects
   - Prefer tuples over lists for fixed data

4. **Error Handling / 错误处理**
   - Use custom exception classes for domain errors
   - Always include context in error messages (EN+CN)
   - Use `raise ... from ...` for exception chaining

```python
class ParseError(Exception):
    """Raised when parsing fails (解析失败时抛出)."""
    pass

def parse_value(text: str) -> int:
    """Parse integer from text (从文本解析整数)."""
    try:
        return int(text)
    except ValueError as e:
        raise ParseError(f"Invalid integer: {text} (无效的整数)") from e
```

5. **Testing / 测试规范**
   - Test function names: `test_<function_name>_<scenario>`
   - Use type-safe fixtures with annotations
   - Include docstrings in tests (EN+CN)

```python
def test_calculate_average_empty_list() -> None:
    """Test that empty list raises ValueError.
    
    测试空列表抛出 ValueError。
    """
    with pytest.raises(ValueError):
        calculate_average([])
```

### Project Config

- **Topology**: hierarchical-mesh
- **Max Agents**: 15
- **Memory**: hybrid
- **HNSW**: Enabled
- **Neural**: Enabled

## Build & Test

**必须使用 uv 管理项目和运行测试 (MUST use uv for project management and testing)**

```bash
# Install dependencies
uv sync

# Run all tests
make test

# Or run tests directly with uv
uv run pytest -v --tb=short

# Run all checks (lint + typecheck + test)
make check

# Individual checks
make lint       # ruff check
make format     # ruff format
make typecheck  # mypy
```

**必须使用 Makefile 跑全测试 (MUST use Makefile for full test suite)**

| Command | Description |
|---------|-------------|
| `make test` | Run pytest with uv |
| `make check` | Run lint + typecheck + test |
| `make lint` | Run ruff linter |
| `make format` | Run ruff formatter |
| `make typecheck` | Run mypy type checker |
| `make fix` | Auto-fix lint issues and format |

- ALWAYS run `make test` after making code changes
- ALWAYS run `make check` before committing

## Security Rules

- NEVER hardcode API keys, secrets, or credentials in source files
- NEVER commit .env files or any file containing secrets
- Always validate user input at system boundaries
- Always sanitize file paths to prevent directory traversal
- Run `npx @claude-flow/cli@latest security scan` after security-related changes

## Concurrency: 1 MESSAGE = ALL RELATED OPERATIONS

- All operations MUST be concurrent/parallel in a single message
- Use Claude Code's Task tool for spawning agents, not just MCP
- ALWAYS batch ALL todos in ONE TodoWrite call (5-10+ minimum)
- ALWAYS spawn ALL agents in ONE message with full instructions via Task tool
- ALWAYS batch ALL file reads/writes/edits in ONE message
- ALWAYS batch ALL Bash commands in ONE message

## Swarm Orchestration

- MUST initialize the swarm using CLI tools when starting complex tasks
- MUST spawn concurrent agents using Claude Code's Task tool
- Never use CLI tools alone for execution — Task tool agents do the actual work
- MUST call CLI tools AND Task tool in ONE message for complex work

### 3-Tier Model Routing (ADR-026)

| Tier | Handler | Latency | Cost | Use Cases |
|------|---------|---------|------|-----------|
| **1** | Agent Booster (WASM) | <1ms | $0 | Simple transforms (var→const, add types) — Skip LLM |
| **2** | Haiku | ~500ms | $0.0002 | Simple tasks, low complexity (<30%) |
| **3** | Sonnet/Opus | 2-5s | $0.003-0.015 | Complex reasoning, architecture, security (>30%) |

- Always check for `[AGENT_BOOSTER_AVAILABLE]` or `[TASK_MODEL_RECOMMENDATION]` before spawning agents
- Use Edit tool directly when `[AGENT_BOOSTER_AVAILABLE]`

## Swarm Configuration & Anti-Drift

- ALWAYS use hierarchical topology for coding swarms
- Keep maxAgents at 6-8 for tight coordination
- Use specialized strategy for clear role boundaries
- Use `raft` consensus for hive-mind (leader maintains authoritative state)
- Run frequent checkpoints via `post-task` hooks
- Keep shared memory namespace for all agents

```bash
npx @claude-flow/cli@latest swarm init --topology hierarchical --max-agents 8 --strategy specialized
```

## Swarm Execution Rules

- ALWAYS use `run_in_background: true` for all agent Task calls
- ALWAYS put ALL agent Task calls in ONE message for parallel execution
- After spawning, STOP — do NOT add more tool calls or check status
- Never poll TaskOutput or check swarm status — trust agents to return
- When agent results arrive, review ALL results before proceeding

## V3 CLI Commands

### Core Commands

| Command | Subcommands | Description |
|---------|-------------|-------------|
| `init` | 4 | Project initialization |
| `agent` | 8 | Agent lifecycle management |
| `swarm` | 6 | Multi-agent swarm coordination |
| `memory` | 11 | AgentDB memory with HNSW search |
| `task` | 6 | Task creation and lifecycle |
| `session` | 7 | Session state management |
| `hooks` | 17 | Self-learning hooks + 12 workers |
| `hive-mind` | 6 | Byzantine fault-tolerant consensus |

### Quick CLI Examples

```bash
npx @claude-flow/cli@latest init --wizard
npx @claude-flow/cli@latest agent spawn -t coder --name my-coder
npx @claude-flow/cli@latest swarm init --v3-mode
npx @claude-flow/cli@latest memory search --query "authentication patterns"
npx @claude-flow/cli@latest doctor --fix
```

## Available Agents (60+ Types)

### Core Development
`coder`, `reviewer`, `tester`, `planner`, `researcher`

### Specialized
`security-architect`, `security-auditor`, `memory-specialist`, `performance-engineer`

### Swarm Coordination
`hierarchical-coordinator`, `mesh-coordinator`, `adaptive-coordinator`

### GitHub & Repository
`pr-manager`, `code-review-swarm`, `issue-tracker`, `release-manager`

### SPARC Methodology
`sparc-coord`, `sparc-coder`, `specification`, `pseudocode`, `architecture`

## Memory Commands Reference

```bash
# Store (REQUIRED: --key, --value; OPTIONAL: --namespace, --ttl, --tags)
npx @claude-flow/cli@latest memory store --key "pattern-auth" --value "JWT with refresh" --namespace patterns

# Search (REQUIRED: --query; OPTIONAL: --namespace, --limit, --threshold)
npx @claude-flow/cli@latest memory search --query "authentication patterns"

# List (OPTIONAL: --namespace, --limit)
npx @claude-flow/cli@latest memory list --namespace patterns --limit 10

# Retrieve (REQUIRED: --key; OPTIONAL: --namespace)
npx @claude-flow/cli@latest memory retrieve --key "pattern-auth" --namespace patterns
```

## Quick Setup

```bash
claude mcp add claude-flow -- npx -y @claude-flow/cli@latest
npx @claude-flow/cli@latest daemon start
npx @claude-flow/cli@latest doctor --fix
```

## Claude Code vs CLI Tools

- Claude Code's Task tool handles ALL execution: agents, file ops, code generation, git
- CLI tools handle coordination via Bash: swarm init, memory, hooks, routing
- NEVER use CLI tools as a substitute for Task tool agents

## Support

- Documentation: https://github.com/ruvnet/claude-flow
- Issues: https://github.com/ruvnet/claude-flow/issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nerdneilsfield)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nerdneilsfield)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
