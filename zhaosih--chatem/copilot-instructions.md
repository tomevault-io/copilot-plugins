## chatem

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CHATEM (AI-Collaborative Engineering Assistant) is a Python-based automation framework for electromagnetic field simulations. It integrates multiple components to provide a unified AI-controllable interface for simulation software.

### Core Architecture

```
User Chat Interface (localhost:3001)
        ↓
AgentAPI (localhost:8080) - Controls AI Agent (Claude/Gemini)
        ↓
MCP Tools - Exposes simulation capabilities
        ↓
Celery Tasks (Redis-backed) - Executes simulations
        ↓
CST Studio Suite - Electromagnetic simulation software
```

## Configuration System

CHATEM uses a flexible YAML-based configuration system with environment variable overrides.

### Configuration Files

- **Primary Config**: `config/chatem.yaml`
- **Local Overrides**: `config/chatem.local.yaml` (gitignored)
- **Environment**: `.env.local` (gitignored)

### Message Queue Configuration

**默认配置 (推荐用于工程师)**:
```yaml
message_queue:
  type: sqlite  # 无需额外安装，开箱即用
```

**企业配置 (高性能需求)**:
```yaml
message_queue:
  type: redis  # 需要安装 Redis 服务器
```

### Key Configuration Sections

```yaml
# 消息队列配置 (SQLite 默认，Redis 可选)
message_queue:
  type: sqlite  # sqlite 或 redis
  sqlite:
    broker_db: data/celery_broker.db
    result_db: data/celery_results.db
  redis:
    host: localhost
    port: 6379
    db: 0

# Agent API Configuration  
agent_api:
  port: 8080
  agent_type: gemini  # claude, gemini, aider, goose

# Frontend UI Configuration  
ui:
  port: 3001
  agent_api_url: http://localhost:8080
  flower_api_url: http://localhost:5555

# Celery Configuration (自动根据 message_queue.type 设置)
celery:
  broker_url: sqlite:///data/celery_broker.db  # SQLite 模式
  # broker_url: redis://localhost:6379/0       # Redis 模式
  result_backend: sqlite:///data/celery_results.db
  pool: solo  # Windows compatibility

# System Configuration
system:
  log_level: INFO
  debug: false
  max_workers: 2
```

## Common Commands

### Configuration Management
```bash
# Load configuration in Python
from src.core.config import get_config
config = get_config()

# Override with environment variables
export MESSAGE_QUEUE_TYPE=redis  # 切换到Redis模式
export AGENT_TYPE=claude
export LOG_LEVEL=DEBUG

# 企业用户快速切换到Redis
# 1. 修改 config/chatem.yaml 中 message_queue.type: redis
# 2. 或设置环境变量: set MESSAGE_QUEUE_TYPE=redis
```

### Service Management
```bash
# Start all services (自动检测消息队列类型)
python scripts/start_services.py

# Start with custom configuration
export MESSAGE_QUEUE_TYPE=redis
export AGENT_TYPE=claude
python scripts/start_services.py

# Alternative console command (after pip install)
chatem                    # Start all services
```

### Dependencies

### Core Requirements
- Python 3.9-3.12 (64-bit Windows)
- **SQLite** (默认，Python内置，无需安装)
- **Redis Server** (可选，企业用户高性能需求)
- CST Studio Suite 2025 (for simulations)

### For Engineers (推荐)
- ✅ Python + npm (已有)
- ✅ 无需额外安装Redis
- ✅ 开箱即用

### For Enterprise Users (可选)
- Redis Server (configurable host/port)
- 修改配置文件切换到Redis模式

### Frontend Development
```bash
# Start UI development server
cd chatem_ui
bun run dev

# Build for production
bun run build

# Configure API endpoints
# In .env.local:
NEXT_PUBLIC_AGENT_API_URL=http://localhost:8080
NEXT_PUBLIC_FLOWER_API_URL=http://localhost:5555
```

### Testing
```bash
# Run all tests
pytest tests/

# Run unit tests only
pytest tests/unit/ -m "not cst"

# Run with configuration
export LOG_LEVEL=DEBUG
pytest tests/ --cov=src --cov-report=html
```

## Service Architecture

### Core Services

1. **AgentAPI Server** (`localhost:8080`)
   - Controls AI agents (Claude, Gemini, Aider, Goose)
   - Provides HTTP API and SSE events
   - Configured via `config.agent_api`

2. **Celery Task Queue** (`redis://localhost:6379/0`)
   - Background simulation processing
   - Distributed task execution
   - Configured via `config.celery`

3. **Flower Monitoring** (`localhost:5555`)
   - Celery task monitoring web interface
   - REST API for task/worker status
   - Configured via `config.flower`

4. **Frontend UI** (`localhost:3001`)
   - React-based chat interface
   - Real-time monitoring dashboard
   - Configured via `config.ui`

### Service Dependencies

```
Redis (required) → Celery Workers → MCP Tools → CST Studio Suite
                ↓
              Flower API ← Frontend Monitoring
                ↓
              AgentAPI ← Frontend Chat
```

## MCP Tools Integration

The system exposes simulation capabilities through MCP (Model Context Protocol) tools:

### IMPORTANT: Direct MCP Tool Usage
**When working with CHATEM's CST tools, ALWAYS call MCP tools directly:**
- ✅ **Correct**: `mcp__cst-simulation__cst_create_brick(...)`
- ❌ **Wrong**: `claude "use cst_create_bridge to create..."` 

The system uses stdio transport, so direct function calls are preferred over chat-based indirect calls.

### MCP Tool Permissions
**Grant batch permissions for all CHATEM CST simulation tools:**
```
Allow all tools from mcp__cst-simulation__ server including:
- Connection: cst_connect, cst_disconnect
- Projects: cst_open_project, cst_close_project, cst_save_project, cst_get_project_info
- Parameters: cst_set_parameter, cst_get_parameter
- Geometry: cst_create_brick, cst_create_cylinder, cst_create_sphere
- Materials: cst_create_material, cst_get_material_list
- Simulation: cst_run_simulation, cst_start_simulation, cst_stop_simulation
- Results: cst_get_s_parameters, cst_get_all_s_parameters
- All other CST tools for efficient workflow
```

### CST Tools
- `cst_open_project` - Open CST project files
- `cst_set_parameter` - Set simulation parameters
- `cst_run_simulation` - Execute simulations
- `cst_get_s_parameters` - Extract results

### CST Parameter Sweep Tools ✨ NEW
- `cst_parameter_sweep_open` - Automatically open Parameter Sweep dialog
- `cst_parameter_sweep_add_parameter` - Add parameter to sweep configuration
- `cst_parameter_sweep_start` - Start Parameter Sweep execution  
- `cst_parameter_sweep_close` - Close Parameter Sweep dialog
- `cst_parameter_sweep_status` - Get Parameter Sweep status and configuration

### CST Optimizer Tools ✨ NEW
- `cst_optimizer_open` - Automatically open Optimizer dialog
- `cst_optimizer_set_objective` - Set optimization objective function
- `cst_optimizer_add_parameter` - Add parameter to optimization configuration
- `cst_optimizer_start` - Start optimization process
- `cst_optimizer_close` - Close Optimizer dialog
- `cst_optimizer_status` - Get Optimizer status and configuration

### CST History List Tools ✨ NEW
- `cst_history_list_get_items` - **Complete workflow**: Auto-opens dialog + extracts all history items ⭐
- `cst_history_list_open` - Manually open History List dialog via Modeling menu
- `cst_history_list_select_item` - Select specific history record item
- `cst_history_list_get_vba_code` - Get VBA code for specific history item ⭐
- `cst_history_list_export_all_vba` - Export VBA code for all history items ⭐
- `cst_history_list_close` - Close History List dialog
- `cst_history_list_status` - Get History List status and configuration

### AI Parameter Sweep Tools 🚀 REVOLUTIONARY
**替代CST原生Parameter Sweep的智能解决方案**
- `ai_parameter_sweep_initialize` - Initialize AI Parameter Sweep engine
- `ai_parameter_sweep_create_config` - Create multi-group parameter sweep configuration
- `ai_parameter_sweep_create_simple_config` - Create simple single-parameter sweep
- `ai_parameter_sweep_create_antenna_config` - Create typical antenna parameter sweep
- `ai_parameter_sweep_prepare_tasks` - Prepare simulation tasks with optimization
- `ai_parameter_sweep_execute` - Execute complete parameter sweep with smart scheduling
- `ai_parameter_sweep_get_status` - Monitor sweep progress in real-time
- `ai_parameter_sweep_get_results_summary` - Get comprehensive results analysis
- `ai_parameter_sweep_get_usage_guide` - Get complete usage documentation

**AI Parameter Sweep 核心优势:**
- 🚫 **无需结果图Tab限制** - 绕过CST原生界面约束
- 🔄 **多重循环扫描** - 支持多组参数，每组多个参数，每个参数多个步骤
- 🧠 **智能执行调度** - 三种策略：顺序、随机、优化排序
- 📊 **自动结果抓取** - 1D/2D结果自动保存，支持多种格式
- ⚡ **乱序执行支持** - 避免系统性偏差，提高执行效率
- 📈 **完整进度跟踪** - 实时监控，详细执行报告
- 💾 **结构化存储** - JSON + Touchstone + CSV 多格式输出

### Celery Management Tools
- `task_get_status` - Check task status
- `task_cancel` - Cancel running tasks
- `task_get_queue_status` - Monitor task queue
- `worker_get_status` - Check worker status

### System Monitoring Tools
- `get_system_status` - Overall system health
- `get_resource_usage` - Resource monitoring

## Usage Patterns

### Basic Simulation Workflow
```python
# Via MCP tools (called by AI agent)
1. cst_open_project("path/to/project.cst")
2. cst_set_parameter("ridge_gap", 2.5)
3. cst_run_simulation_async()  # Returns task_id
4. task_get_status(task_id)    # Monitor progress
5. cst_get_s_parameters()      # Extract results
```

### Parameter Sweep Workflow ✨ NEW
```python
# Automated Parameter Sweep via MCP tools
1. cst_parameter_sweep_open()                           # Open Parameter Sweep dialog
2. cst_parameter_sweep_add_parameter("gap", 1.0, 5.0, 5)  # Add parameter: gap from 1-5mm, 5 steps
3. cst_parameter_sweep_add_parameter("width", 10, 20, 3)   # Add parameter: width from 10-20mm, 3 steps  
4. cst_parameter_sweep_start()                          # Start sweep execution
5. cst_parameter_sweep_status()                         # Monitor sweep status
6. cst_parameter_sweep_close()                          # Close when complete
```

### Optimizer Workflow ✨ NEW
```python
# Automated Optimization via MCP tools
1. cst_optimizer_open()                                 # Open Optimizer dialog
2. cst_optimizer_set_objective("minimize(S11)")        # Set objective function
3. cst_optimizer_add_parameter("gap", 1.0, 5.0)        # Add parameter: gap from 1-5mm
4. cst_optimizer_add_parameter("width", 10, 20)        # Add parameter: width from 10-20mm
5. cst_optimizer_start()                                # Start optimization
6. cst_optimizer_status()                               # Monitor optimization status
7. cst_optimizer_close()                                # Close when complete
```

### History List & VBA Export Workflow ✨ NEW
```python
# Complete automated History List extraction (one-step solution)
1. cst_history_list_get_items()                         # ⭐ Complete workflow: Opens + extracts all items
   # This automatically:
   # - Finds CST window
   # - Opens History List dialog
   # - Uses keyboard navigation to read all blocks
   # - Saves data to JSON file
   # - Returns all history items

# Optional: Work with specific items
2. cst_history_list_get_vba_code("Create Brick")        # Get VBA for specific operation
3. cst_history_list_export_all_vba()                    # Export ALL VBA code at once ⭐
4. cst_history_list_close()                             # Close when complete
```

### AI Parameter Sweep Workflow 🚀 REVOLUTIONARY
```python
# Intelligent AI-Controlled Parameter Sweep (bypasses CST GUI limitations)
1. ai_parameter_sweep_initialize()                      # Initialize AI engine
2. ai_parameter_sweep_create_antenna_config()           # Create antenna sweep config
   # OR create custom config:
   # ai_parameter_sweep_create_config(
   #     project_name="Multi-Parameter Sweep",
   #     parameter_groups=[{
   #         "name": "Antenna Dimensions", 
   #         "parameters": [
   #             {"name": "patch_length", "min_value": 25, "max_value": 35, "steps": 5},
   #             {"name": "patch_width", "min_value": 35, "max_value": 45, "steps": 5}
   #         ]
   #     }],
   #     execution_strategy="optimized"  # sequential, random, optimized
   # )
3. ai_parameter_sweep_prepare_tasks()                   # Generate optimized task sequence
4. ai_parameter_sweep_execute()                         # Execute with smart scheduling
5. ai_parameter_sweep_get_status()                      # Monitor progress (25 tasks, 60% complete)
6. ai_parameter_sweep_get_results_summary()             # Get comprehensive analysis

# Results automatically saved to:
# - sweep_results/task_xxx/parameters.json (parameter sets)
# - sweep_results/task_xxx/s_parameters.json (S-parameter data)  
# - sweep_results/task_xxx/results.s2p (Touchstone files)
# - sweep_results/task_xxx/farfield_*.json (Far-field data)
# - sweep_results/sweep_report.json (execution summary)
```

**AI Parameter Sweep vs Traditional:**
- ❌ **Traditional**: Requires result tab, GUI interaction, limited parameters
- ✅ **AI-Controlled**: No GUI limits, multi-group parameters, intelligent scheduling
- ❌ **Traditional**: Manual result collection, prone to errors
- ✅ **AI-Controlled**: Automatic 1D/2D result extraction, structured storage
- ❌ **Traditional**: Sequential execution only
- ✅ **AI-Controlled**: Random/optimized execution, reduced parameter change overhead

### Configuration Management
```python
# Get current configuration
from src.core.config import get_config
config = get_config()

# Access configuration values
agent_port = config.agent_api.port
redis_url = config.celery.broker_url
log_level = config.system.log_level

# Reload configuration
config_manager.reload_config()
```

### Task Management
```python
# Submit background task
from src.tasks.celery_app import get_celery_app
celery_app = get_celery_app()
result = celery_app.send_task('simulation_task', args=[param1, param2])

# Monitor via Flower API
import requests
workers = requests.get(f"http://localhost:{config.flower.port}/api/workers")
```

## Dependencies

### Core Requirements
- Python 3.9-3.12 (64-bit Windows)
- **SQLite** (默认，Python内置，无需安装)
- **Redis Server** (可选，企业用户高性能需求)
- CST Studio Suite 2025 (for simulations)

### For Engineers (推荐)
- ✅ Python + npm (已有)
- ✅ 无需额外安装Redis
- ✅ 开箱即用

### For Enterprise Users (可选)
- Redis Server (configurable host/port)
- 修改配置文件切换到Redis模式

### Key Python Packages
- **Configuration**: pyyaml, dataclasses
- **Web/API**: FastAPI, uvicorn, pydantic
- **Task Queue**: celery, redis (可选), flower
- **MCP Integration**: fastmcp or mcp-sdk
- **Frontend**: Next.js, React, TypeScript

## Development Notes

### Configuration Priority
1. **Environment Variables** (highest priority)
2. **Local Config Files** (`config/*.local.yaml`)
3. **Main Config File** (`config/chatem.yaml`)
4. **Default Values** (lowest priority)

### Service Ports
- **Frontend UI**: 3001 (configurable via `config.ui.port`)
- **AgentAPI**: 8080 (configurable via `config.agent_api.port`)
- **Flower**: 5555 (configurable via `config.flower.port`)
- **Redis**: 6379 (configurable via `config.redis.port`)

### Development Workflow
1. Edit configuration in `config/chatem.yaml` or use environment variables
2. Start services: `python scripts/start_services.py`
3. Access UI at configured port (default: http://localhost:3001)
4. Chat with AI agent to control simulations
5. Monitor tasks via Flower interface

### Environment-Specific Configuration

**Development**:
```yaml
system:
  debug: true
  log_level: DEBUG
agent_api:
  agent_type: gemini
```

**Production**:
```bash
export DEBUG=false
export LOG_LEVEL=ERROR
export AGENT_TYPE=claude
```

## File Structure

```
├── config/
│   ├── chatem.yaml              # Main configuration
│   ├── chatem.local.yaml        # Local overrides (gitignored)
│   └── callback_prompts.yaml    # AI callback templates
├── src/
│   ├── core/
│   │   ├── config.py           # Configuration management
│   │   └── prompt_management.py # AI prompt templates
│   ├── agent/
│   │   └── fastmcp_server.py   # MCP tools implementation
│   ├── tasks/
│   │   └── celery_app.py       # Celery task definitions
│   └── cst_native/
│       └── controller.py       # CST integration
├── chatem_ui/                  # Frontend React application
├── scripts/
│   └── start_services.py       # Service launcher
└── docs/
    └── configuration.md        # Configuration documentation
```

## Important Notes

- All services use configuration from `config/chatem.yaml`
- Environment variables override configuration file values
- Local configuration files (`.local.yaml`) are gitignored
- The system is designed for Windows + CST Studio Suite
- All ports and URLs are configurable
- Context managers ensure proper resource cleanup
- The system supports multiple AI agents (Claude, Gemini, Aider, Goose)

## Future Extensibility

- **Multi-Software Support**: Designed to support HFSS, ADS, and other simulation software
- **Configuration Flexibility**: Easy to add new configuration sections
- **Agent Agnostic**: Can work with any AI agent that supports MCP
- **Distributed Processing**: Celery supports scaling across multiple machines
- **Plugin Architecture**: MCP tools can be extended for new simulation software

## CST Studio Suite Integration

### Official Documentation Access
- **Python API Documentation**: `C:\Program Files (x86)\CST Studio Suite 2025\Online Help\Python`
- User has granted access to read CST official documentation for API reference

### Allowed Directories and Operations
- CST documentation directory: Read access granted for API reference
- Add other user-approved directories and operations here as needed

### Key Integration Notes
- **CST connect function now connects to existing opened projects by default** ✅
- This matches simulation engineers' typical workflow patterns
- Uses CST's official `connect_to_any_or_new()` API method
- If user has CST open with a project, it connects to that existing environment
- Only creates new environment if no CST is running

---
> Source: [zhaosih/ChatEM](https://github.com/zhaosih/ChatEM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
