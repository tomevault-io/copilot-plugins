## agentdemo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a professional ERP AI Agent demonstration project showcasing modern enterprise-grade UI/UX with LLM-powered intelligent automation. The project consists of:

1. **Professional ERP System**: Complete web application with modern UI/UX design, sidebar navigation, and real-time dashboards
2. **LLM AI Agent**: Intelligent agent using Ollama local language models for natural language understanding
3. **Realistic Data**: 27 products, 25 historical orders, 6 customer companies with complete information

The project demonstrates enterprise-level web design combined with intelligent AI automation.

## Architecture

### System Components

**ERP System** ([erp-system/](erp-system/))
- Backend: FastAPI REST API ([backend/main.py](erp-system/backend/main.py:1))
- Database: SQLite with SQLAlchemy ORM ([backend/database.py](erp-system/backend/database.py:1))
- Frontend: Static HTML pages served by FastAPI ([frontend/](erp-system/frontend/))
- Key features: Order management, inventory tracking, sales/inventory reporting

**AI Agents** ([agent/](agent/))
- Rule-based agent ([agent.py](agent/agent.py:1)): Uses regex patterns via TaskParser
- LLM agent ([llm_agent.py](agent/llm_agent.py:1)): Uses Ollama with function calling
- Both agents communicate with ERP via HTTP REST API

### Data Flow

1. User provides natural language command to agent
2. Agent parses intent (rule-based: regex patterns, LLM: language model inference)
3. Agent calls appropriate ERP API endpoints
4. Agent formats and displays results to user

### Key Architectural Patterns

**Agent Architecture** (LLM version):
- Function calling/tool use pattern for API interactions
- Stateless request-response model (no conversation memory beyond single turn)
- Declarative tool definitions with parameters in [llm_agent.py:32-83](agent/llm_agent.py:32-83)
- Product ID caching to reduce API calls

**ERP Backend**:
- RESTful API design with Pydantic models for validation
- Database session management via dependency injection (FastAPI Depends)
- Automatic database initialization with realistic data on startup
- Inventory management: automatic stock updates on order creation/cancellation
- Auto-generated order numbers (format: ORD2410xxxx)

**ERP Frontend**:
- Professional UI/UX with sidebar navigation and top navbar
- Shared CSS file (style.css) for consistent styling
- Color scheme: Dark blue-grey primary (#2c3e50), with functional colors
- Responsive design supporting mobile/tablet
- Real-time data updates via REST API
- Pages: Dashboard (index.html), Orders, Inventory, Reports, AI Chat

## Development Commands

### Starting the System

**Start ERP system** (required first):
```bash
./start_erp.sh
# Or manually:
cd erp-system/backend && python3 main.py
```
Access at http://localhost:8000 (web UI) or http://localhost:8000/docs (API docs)

**Start LLM Agent**:
```bash
./start_llm_agent.sh
# Or manually (specify model):
cd agent && python3 llm_agent.py qwen2.5:7b
```

Note: Rule-based agent (agent.py) has been removed. Only LLM agent is available.

### Ollama Setup (for LLM Agent)

Install Ollama:
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start Ollama service:
```bash
ollama serve
```

Download recommended model:
```bash
ollama pull qwen2.5:7b
```

See [OLLAMA_SETUP.md](OLLAMA_SETUP.md:1) for detailed instructions and model recommendations.

### Dependencies

Install ERP system dependencies:
```bash
cd erp-system/backend && pip install -r requirements.txt
```

Install agent dependencies:
```bash
cd agent && pip install -r requirements.txt
```

### Database Management

**Database file**: `erp-system/backend/erp_demo.db` (auto-created on first run)

**Reset database**: Delete the database file, it will be recreated with realistic data:
```bash
rm erp-system/backend/erp_demo.db
```

Database initialization includes:
- 27 realistic products (Dell, HP, Lenovo, ASUS, Logitech, etc.) with SKU, category, cost, supplier
- 6 Taiwan companies with complete contact information
- 25 historical orders from past 3 months with auto-generated order numbers

## API Endpoints

All endpoints are prefixed with `/api/`:

**Products**: GET/POST/PUT/DELETE `/api/products[/{id}]`
**Orders**: GET/POST/PUT/DELETE `/api/orders[/{id}]`
**Inventory**: GET `/api/inventory/alerts`, POST `/api/inventory/restock/{id}`
**Reports**: GET `/api/reports/sales`, GET `/api/reports/inventory`

Full API documentation available at http://localhost:8000/docs when ERP is running.

## Agent Implementation Details

### LLM Agent Tool Definitions

The LLM agent uses a function calling pattern. Tools are defined in [llm_agent.py:33-83](agent/llm_agent.py:33-83) with:
- Tool name and description
- Parameter specifications
- Implementation methods (tool_*)

**Adding new tools**:
1. Add tool definition to `self.tools` list
2. Implement `tool_{name}()` method
3. Add handler to `tool_map` in `execute_tool()`

### Rule-based Agent Patterns

The rule-based agent uses [task_parser.py](agent/task_parser.py:1) which matches commands against predefined regex patterns. To add new command patterns, modify TaskParser class methods.

### Product ID Mapping

Products have fixed IDs (defined in database initialization):
1. 笔记本电脑 (Laptop)
2. 台式电脑 (Desktop)
3. 显示器 (Monitor)
4. 键盘 (Keyboard)
5. 鼠标 (Mouse)
6. 打印机 (Printer)

The LLM agent has this mapping in its system prompt ([llm_agent.py:148](agent/llm_agent.py:148)).

## Configuration

**ERP API URL**: Default `http://localhost:8000/api` (configurable in agent `__init__`)
**Ollama API URL**: Default `http://localhost:11434` (configurable in LLM agent `__init__`)
**Default LLM Model**: `qwen2.5:7b` (changeable via command-line argument or in [llm_agent.py:533](agent/llm_agent.py:533))

## Testing Scenarios

Example commands to test the LLM agent:
- "帮我给ABC公司下个订单，买10台笔记本电脑"
- "查一下哪些产品库存不够了"
- "给库存最少的产品补货50个"
- "把1号订单标记为完成"
- "告诉我这个月的销售情况"

## Common Issues

**"无法连接到ERP系统"**: Ensure ERP is running at http://localhost:8000
**"无法连接到Ollama服务"**: Run `ollama serve` in separate terminal
**"模型未找到"**: Download model with `ollama pull qwen2.5:7b`
**Port 8000 already in use**: Kill existing process or change port in [main.py:305](erp-system/backend/main.py:305)

## Technology Stack

**ERP Backend**: FastAPI, SQLAlchemy, Pydantic, Uvicorn, SQLite
**ERP Frontend**: HTML, JavaScript, Bootstrap (vanilla, no frameworks)
**Agents**: Python, Requests, Colorama, Ollama (LLM version only)
**Recommended Python version**: 3.8+

---
> Source: [chrisw1005/AgentDemo](https://github.com/chrisw1005/AgentDemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
