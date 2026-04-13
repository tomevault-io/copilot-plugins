## witsv3

> - **Always read `PLANNING.md`** at the start of new conversations to understand WitsV3's LLM wrapper architecture and goals

### 🔄 Project Context & Architecture

- **Always read `PLANNING.md`** at the start of new conversations to understand WitsV3's LLM wrapper architecture and goals
- **Check `TASK.md`** before starting work. Add tasks if not listed with date
- **WitsV3 is an LLM orchestration system** with CLI-first approach, ReAct pattern, and tool registry
- **Core components**: agents/, core/, tools/ - maintain this structure

### 🐍 Python & Async Best Practices

- **Use Python 3.10+** with type hints throughout
- **All agent/tool methods must be async** - WitsV3 is fully async
- **Follow PEP8** and format with black
- **Use Pydantic models** for data validation (already in core/schemas.py)
- **Import style**: Use relative imports within packages, absolute for cross-package

### 🧱 Code Structure & Modularity

- **Max 500 lines per file** - split into modules when approaching limit
- **Maintain existing structure**:
  - `agents/` - Agent implementations (BaseAgent, Orchestrators, Control Center)
  - `core/` - Core functionality (config, LLM interface, memory, schemas)
  - `tools/` - Tool implementations extending BaseTool
- **New tools** go in tools/ and must extend BaseTool
- **New agents** go in agents/ and must extend BaseAgent

### 🧪 Testing Requirements

- **Create pytest tests for all new features** in `/tests` mirroring structure
- **Test async functions** with pytest-asyncio
- **Mock external services** (Ollama, file system) in tests
- **Minimum coverage per feature**:
  - Happy path test
  - Error handling test
  - Edge case test
- **Run existing test functions** at bottom of modules when updating

### ✅ Task Management

- **Update TASK.md** after completing tasks
- **Add discovered work** to TASK.md under "Discovered During Work"
- **Log progress** in task descriptions with dates

### 📚 WitsV3-Specific Conventions

- **StreamData for agent communication** - use stream_thinking(), stream_action(), etc.
- **Tool Registry pattern** - all tools must register with get_llm_description()
- **Memory Manager integration** - use store_memory() and search_memory() in agents
- **Config-driven** - use WitsV3Config for all settings
- **Logging pattern**: Use self.logger in all classes

### 🧠 LLM Integration Rules

- **Ollama is primary LLM** - ensure compatibility
- **Support streaming responses** via AsyncGenerator
- **Parse LLM responses robustly** using ResponseParser
- **Handle tool calls** through proper ToolCall/ToolResult schemas

### 📝 Documentation Standards

- **Update README.md** when adding features or changing setup
- **Google-style docstrings** for all functions:
  ```python
  async def method_name(self, param: str) -> StreamData:
      """
      Brief description of method.

      Args:
          param: Description of parameter

      Yields:
          StreamData objects for streaming responses
      """
  ```
- **Comment non-obvious logic** with # Reason: explanations

### ⚠️ Critical WitsV3 Rules

- **Never break async patterns** - all I/O operations must be async
- **Preserve ReAct loop** in orchestrators (Reason-Act-Observe)
- **Maintain backward compatibility** with existing tools/agents
- **Test with Ollama running** - many features require active Ollama connection
- **Unicode handling** is important - project emphasizes clean text output
- **Memory persistence** - ensure MemorySegment serialization works

### 🚫 Don't Do

- Don't use synchronous I/O operations
- Don't create circular imports between packages
- Don't modify core schemas without updating all dependent code
- Don't hardcode configuration values - use config.yaml
- Don't forget to handle LLM connection failures gracefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/capnknives) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
