## swordswarm

> SWORDSwarm is a production-ready multi-agent AI orchestration system with 88+ specialized agents, hardware acceleration, and comprehensive testing infrastructure.

# Cursor AI Rules for SWORDSwarm v42.0

## Project Context
SWORDSwarm is a production-ready multi-agent AI orchestration system with 88+ specialized agents, hardware acceleration, and comprehensive testing infrastructure.

## Architecture Overview
- **Three-tier Architecture:**
  - Agent Layer: Python 3.11+ agent implementations
  - Hook Layer: C/Rust acceleration primitives
  - Binary Layer: Hardware-optimized operations (NPU, AVX-512, SIMD)
- **Agent Hierarchy:** Executive → Management → Team → Specialist
- **Hardware Acceleration:** 7-10x speedup with Intel NPU and AVX optimization

## Code Style & Standards
- Python: Follow PEP 8, max line length 100 characters
- Use Black formatter and isort for imports
- Type hints required for all function signatures
- Maintain test coverage above 80%
- All async functions must use asyncio properly

## Agent Development Guidelines
1. **Agent Structure:** Inherit from BaseAgent in `agents/src/python/claude_agents/base_agent.py`
2. **Agent Registry:** Register new agents in `config/.claude.json`
3. **Testing:** Create corresponding test in `tests/test_<agent_name>.py`
4. **Documentation:** Update `docs/ACCURATE_AGENT_MAPPING.md` for new agents
5. **Hardware Awareness:** Use hardware optimization when available via `HardwareOptimizer`

## Critical Patterns
- **Agent Communication:** Use `AgentOrchestrator` for agent coordination
- **Database Access:** Use `DatabaseManager` with connection pooling
- **Hardware Detection:** Check NPU/AVX capabilities before optimization
- **Error Handling:** Use comprehensive try-except with logging
- **Async Operations:** Prefer async/await over threading

## File Organization
```
agents/src/python/claude_agents/    # Agent implementations
hooks/                               # Acceleration hooks
config/                              # Configuration files
tests/                               # Test suite (567+ tests)
docs/                                # Documentation (50+ files)
integration/                         # IDE/tool integrations
```

## Testing Requirements
- Unit tests for all new functions
- Integration tests for agent interactions
- Hardware tests for NPU/AVX features
- Minimum 80% code coverage
- Use pytest with async support

## Performance Considerations
- Enable NPU acceleration when available (7-10x speedup)
- Use AVX-512/AVX2 SIMD for vectorized operations
- Implement connection pooling for database access
- Use async operations for I/O-bound tasks
- Profile before optimizing

## Agent Development Workflow
1. Define agent in `agents/src/python/claude_agents/<agent_name>.py`
2. Register in `config/.claude.json` with tools and capabilities
3. Create tests in `tests/test_<agent_name>.py`
4. Update documentation in `docs/ACCURATE_AGENT_MAPPING.md`
5. Add to orchestration in `integration/claude_unified_integration.py`
6. Test with `pytest tests/test_<agent_name>.py -v`

## Security Guidelines
- Validate all inputs to agents
- Sanitize file paths and shell commands
- Use parameterized queries for database access
- Never commit secrets or API keys
- Follow principle of least privilege

## Integration Points
- **Claude Code SDK:** Use patterns from `integration/claude_unified_integration.py`
- **VSCode/Cursor:** Settings in `.vscode/` and `.cursor/`
- **Warp Terminal:** Workflows in `warp/workflows/`
- **CI/CD:** GitHub Actions in `.github/workflows/`

## Hardware-Specific Notes
- NPU operations: Use OpenVINO runtime
- AVX operations: Detect capability with CPUID
- SIMD vectorization: Use numpy with hardware backends
- Memory alignment: 64-byte alignment for AVX-512

## Common Anti-Patterns to Avoid
- Don't use blocking I/O in async functions
- Don't create agents without tests
- Don't skip hardware capability checks
- Don't use global state in agents
- Don't bypass the agent registry

## Documentation Standards
- All agents must have docstrings with examples
- Update CHANGELOG.md for significant changes
- Keep README.md synchronized with features
- Document all configuration options
- Include performance benchmarks for optimizations

## Claude Proactive Agent System 🤖
**IMPORTANT:** Claude Code now features an extremely proactive agent invocation system!

### Automatic Agent Detection & Invocation
Claude automatically detects your intent and invokes the appropriate agents:
- **Debug/Error requests** → `debugger`, `researcher`, `disassembler`
- **Security concerns** → `security`, `auditor`, `bastion`
- **Performance issues** → `optimizer`, `npu`, `monitor`
- **Testing needs** → `testbed`, `qadirector`
- **Deployment** → `deployer`, `infrastructure`, `security`
- **Architecture/Design** → `architect`, `director`, `apidesigner`
- **Bug fixes** → `debugger`, `patcher`, `researcher`
- **Code quality** → `linter`, `optimizer`, `auditor`

### Event-Based Auto-Invocation
Agents automatically trigger on events:
- File saved → `linter` + language-specific agents
- Test failed → `debugger`, `testbed`
- Build failed → `debugger`, `constructor`
- Git commit → `linter`, `auditor`, `security`
- PR created → `auditor`, `security`

### Configuration
- Enable/disable: `python3 scripts/enable_proactive_agents.py [enable|disable|status]`
- Config file: `.claude/proactive-config.json`
- Documentation: `CLAUDE_PROACTIVE_AGENTS.md`

### Confidence Levels
- ≥70% confidence → Auto-invoke (aggressive mode)
- ≥50% confidence → Suggest to user
- Learns from outcomes and improves over time

## Cursor-Specific Features
- Use `@agent` command to invoke specific agents
- Use `@ask` to query agent capabilities
- Use `@optimize` to apply hardware acceleration
- Use `@test` to run relevant test suites
- Use `@docs` to search documentation

## Quick Reference
- Agent count: 88+ specialized agents
- Test coverage: 82% (567+ tests)
- Languages: Python, C, Rust, TypeScript, Julia, Go
- Hardware: Intel NPU, AVX-512, AVX2, SIMD
- Database: PostgreSQL 16+ with asyncpg
- Python: 3.11+ required

## Need Help?
- Agent docs: `docs/ACCURATE_AGENT_MAPPING.md`
- Integration guide: `config/CLAUDE.md`
- Testing guide: `tests/README.md` (if exists)
- Architecture: `docs/ARCHITECTURE.md` (if exists)

---
> Source: [SWORDOps/SWORDSwarm](https://github.com/SWORDOps/SWORDSwarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
