## headless-pm

> This directory contains Claude Code specific configuration and instructions for working with Headless PM.

# Claude Code Configuration for Headless PM

This directory contains Claude Code specific configuration and instructions for working with Headless PM.

## Quick Start

1. **Ensure Headless PM is running:**
   ```bash
   ./start.sh
   ```

2. **For MCP Integration (Recommended):**
   - Copy `.mcp.json` to your project root
   - Claude Code will automatically detect and use the MCP server
   - Use natural language to interact with Headless PM

3. **For Python Client Integration:**
   - Import the client from `examples/client.py`
   - Follow examples in the client documentation

## MCP Usage Examples

When using MCP, you can use natural language commands:

```
"Register me as a backend developer named claude_001"
"Get my next task"
"Lock task 123"
"Update task 123 status to under_work"
"Create a document about the API implementation"
"Check for any mentions"
```

## Python Client Usage Examples

```python
from examples.client import claude_register, TaskStatus

# Register as an agent
client = claude_register("claude_001", "backend_dev", "senior")

# Get and work on tasks
task = client.get_next_task()
if task:
    client.lock_task(task.id)
    client.update_task_status(task.id, TaskStatus.UNDER_WORK)
    # Do work...
    client.update_task_status(task.id, TaskStatus.DEV_DONE)
```

## Role-Specific Instructions

See the role-specific instructions in:
- `/agents/mcp/` - For MCP-based workflows
- `/agents/client/` - For Python client workflows

## Environment Setup

1. Copy `/agents/env-example` to `.env` in project root
2. Configure your database and API settings
3. Run `./start.sh` to start both API and MCP servers

## Best Practices

1. **Task Management:**
   - Always lock tasks before starting work
   - Update task status as you progress
   - Document your work with the document creation tools

2. **Communication:**
   - Use @mentions in documents to notify team members
   - Check for mentions regularly
   - Create clear, descriptive documents

3. **Service Registration:**
   - Register any microservices you create
   - Send regular heartbeats to maintain service health status

4. **Git Workflow:**
   - Major tasks (complexity: major) require feature branches and PRs
   - Minor tasks can be committed directly to main
   - Follow the project's commit message conventions

## Troubleshooting

- If MCP connection fails, ensure the server is running via `./start.sh`
- For client issues, check that the API is accessible at http://localhost:6969
- Check logs in the terminal where you ran `./start.sh`

## Additional Resources

- API Documentation: http://localhost:6969/api/v1/docs
- CLI Dashboard: `python -m src.cli.main dashboard`
- Project Status: `python -m src.cli.main status`

---
> Source: [madviking/headless-pm](https://github.com/madviking/headless-pm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
