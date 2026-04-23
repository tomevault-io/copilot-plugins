## selfai-npu-agent

> This is a sophisticated **AI-powered terminal chatbot with multi-backend inference support** designed for Windows on ARM with Snapdragon X Elite NPU acceleration. The project implements a three-phase intelligent pipeline (SelfAI) with fallback mechanisms, memory management, and agent-based task execution.

# AI NPU Agent Project - Architecture & Documentation

## Project Overview

This is a sophisticated **AI-powered terminal chatbot with multi-backend inference support** designed for Windows on ARM with Snapdragon X Elite NPU acceleration. The project implements a three-phase intelligent pipeline (SelfAI) with fallback mechanisms, memory management, and agent-based task execution.

**Key Purpose**: Enable efficient local AI inference with automatic fallback from NPU hardware acceleration to CPU execution, all managed through a configuration-driven system with optional planning and merge phases.

---

## Architecture Overview

### High-Level System Design

The system implements a **three-phase pipeline**:

```
┌─────────────────────────────────────────────────────────────────┐
│                        SelfAI Pipeline                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PLANNING PHASE (Ollama-based)                               │
│     ├─ Accepts user goal/request                                │
│     ├─ Generates DPPM plan (Distributed Planning Problem Model) │
│     └─ Creates subtasks with dependencies & merge strategy      │
│                                                                  │
│  2. EXECUTION PHASE (Multi-backend LLM inference)               │
│     ├─ Executes subtasks sequentially/parallel (per plan)       │
│     ├─ Uses AgentManager to route to specialized agents         │
│     ├─ Falls back between backends: AnythingLLM → QNN → CPU    │
│     └─ Saves results and tracks status                          │
│                                                                  │
│  3. MERGE PHASE (Result synthesis)                              │
│     ├─ Collects all subtask outputs                             │
│     ├─ Synthesizes into coherent final answer                   │
│     └─ Falls back gracefully with internal summary              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Backend Inference Strategy

The system supports **three execution backends in priority order**:

1. **AnythingLLM (NPU)** - Primary: Hardware-accelerated inference via Snapdragon X NPU
   - Communicates via HTTP API to AnythingLLM server
   - Configured in `config.yaml` under `npu_provider`
   - Supports streaming output

2. **QNN (Qualcomm Neural Network)** - Secondary: Direct NPU model execution
   - Automatically discovered from `models/` directory
   - Uses QAI Hub models (e.g., Phi-3.5-Mini-Instruct)
   - Optimized for on-device inference

3. **CPU Fallback** - Tertiary: Local CPU inference via llama-cpp-python
   - Uses GGUF quantized models (e.g., Phi-3-mini-4k-instruct.Q4_K_M.gguf)
   - Pure CPU execution, no GPU/NPU required
   - Guarantees functionality even without specialized hardware

**Automatic Failover**: If AnythingLLM fails, system automatically tries QNN, then CPU.

---

## Directory Structure

```
AI_NPU_AGENT_Projekt/
├── CLAUDE.md                          # This file - architecture documentation
├── README.md                           # User-facing project overview
├── UI_GUIDE.md                        # Terminal UI features & customization
├── config.yaml.template               # Configuration template
├── config_extended.yaml               # Extended configuration example
├── .env.example                       # Environment variables template
├── requirements.txt                   # Main dependencies
├── requirements-core.txt              # Core CPU dependencies
├── requirements-npu.txt               # NPU-specific dependencies
│
├── config_loader.py                   # Configuration loading & validation
├── main.py                            # Entry point: Agent initialization
├── llm_chat.py                        # QNN-based chat interface
│
├── selfai/                            # Main SelfAI package
│   ├── __init__.py
│   ├── selfai.py                      # Main CLI loop with full pipeline
│   ├── core/
│   │   ├── agent.py                   # Basic agent with tool-calling loop
│   │   ├── agent_manager.py           # AgentManager: manages multiple agents
│   │   ├── model_interface.py         # Base interface for LLM models
│   │   ├── anythingllm_interface.py   # AnythingLLM HTTP client
│   │   ├── npu_llm_interface.py       # QNN/NPU model interface
│   │   ├── local_llm_interface.py     # CPU fallback (llama-cpp-python)
│   │   ├── planner_ollama_interface.py# Ollama planner client
│   │   ├── merge_ollama_interface.py  # Ollama merge provider
│   │   ├── execution_dispatcher.py    # Subtask execution orchestrator
│   │   ├── memory_system.py           # Conversation & plan storage
│   │   ├── context_filter.py          # Smart context relevance filtering
│   │   ├── planner_validator.py       # Plan schema validation
│   │   └── smolagents_runner.py       # Smolagents integration
│   │
│   ├── tools/
│   │   ├── tool_registry.py           # Tool catalog & management
│   │   ├── filesystem_tools.py        # File/directory operations
│   │   └── shell_tools.py             # Shell command execution
│   │
│   └── ui/
│       └── terminal_ui.py             # Terminal UI with animations
│
├── models/                            # Model storage directory
│   ├── Phi-3-mini-4k-instruct.Q4_K_M.gguf  # CPU fallback model
│   └── [other GGUF/QNN models]
│
├── memory/                            # Conversation & plan storage
│   ├── plans/                         # Saved execution plans
│   └── [memory categories]/           # Memory organized by agent categories
│
├── agents/                            # Agent configurations
│   ├── [agent_key]/
│   │   ├── system_prompt.md           # Agent system prompt
│   │   ├── memory_categories.txt      # Memory categories for this agent
│   │   ├── workspace_slug.txt         # AnythingLLM workspace
│   │   └── description.txt            # Agent description
│   └── [other agents]
│
├── data/                              # Additional data/resources
├── docs/                              # Extended documentation
├── scripts/                           # Setup & utility scripts
├── archive/                           # Old/archived code
└── Learings_aus_Problemen/            # Learning notes & problems
```

---

## Configuration System

### Configuration Loading Flow

```
┌─────────────────────────────────────────────────────────┐
│  config_loader.py::load_configuration()                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Load .env file (secrets)                            │
│  2. Load config.yaml (main settings)                    │
│  3. Normalize config (support both formats)             │
│  4. Resolve environment variables (${VAR_NAME})         │
│  5. Validate required fields                            │
│  6. Create structured dataclasses                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Configuration Structure

**config.yaml** contains:

1. **npu_provider** - AnythingLLM backend
   ```yaml
   npu_provider:
     base_url: "http://localhost:3001/api/v1"
     workspace_slug: "main"
     api_key: "loaded-from-.env"
   ```

2. **cpu_fallback** - Local GGUF model
   ```yaml
   cpu_fallback:
     model_path: "Phi-3-mini-4k-instruct.Q4_K_M.gguf"
     n_ctx: 4096                    # Context window size
     n_gpu_layers: 0                # GPU offload layers
   ```

3. **system** - General settings
   ```yaml
   system:
     streaming_enabled: true        # Enable word-by-word output
     stream_timeout: 60.0           # Streaming timeout in seconds
   ```

4. **agent_config** - Agent management
   ```yaml
   agent_config:
     default_agent: "code_helfer"   # Default agent to load
   ```

5. **planner** - Optional Ollama-based planning
   ```yaml
   planner:
     enabled: false                 # Enable/disable planning
     execution_timeout: 120.0       # Timeout per subtask
     providers:                     # Multiple planner backends
       - name: "local-ollama"
         type: "local_ollama"
         base_url: "http://localhost:11434"
         model: "gemma3:1b"
         timeout: 180.0
         max_tokens: 768
   ```

6. **merge** - Optional result synthesis
   ```yaml
   merge:
     enabled: false
     providers:                     # Multiple merge backends
       - name: "merge-ollama"
         type: "local_ollama"
         base_url: "http://localhost:11434"
         model: "gemma3:3b"
         timeout: 180.0
         max_tokens: 2048
   ```

### Environment Variables

**Required** (in .env file):
- `API_KEY`: AnythingLLM API key (required if using AnythingLLM)

**Optional**:
- `OLLAMA_CLOUD_API_KEY`: For cloud-based Ollama providers
- Any variables referenced in config.yaml as `${VAR_NAME}`

---

## Key Components

### 1. Configuration Loader (`config_loader.py`)

**Purpose**: Centralized, validated configuration management

**Key Classes**:
- `NPUConfig`: AnythingLLM backend settings
- `CPUConfig`: Local model configuration
- `SystemConfig`: General system settings
- `PlannerConfig`: Planning phase configuration
- `MergeConfig`: Merge phase configuration
- `AppConfig`: Complete application config

**Key Functions**:
- `load_configuration()`: Load, validate, and structure config
- `_normalize_config()`: Support both simple and extended formats
- `_resolve_env_template()`: Replace `${VAR_NAME}` with env values

### 2. Agent System (`selfai/core/agent_manager.py`)

**Purpose**: Manage multiple specialized AI agents with memory

**Agent Properties**:
- `key`: Unique identifier (e.g., "code_helfer")
- `display_name`: Human-readable name
- `description`: What this agent does
- `system_prompt`: Agent personality/instructions
- `memory_categories`: Conversation storage categories
- `workspace_slug`: AnythingLLM workspace

**AgentManager Responsibilities**:
- Load agents from disk
- Switch between agents at runtime
- Provide agent to execution/memory systems

### 3. Model Interfaces (Base + Implementations)

**Base**: `ModelInterface` in `model_interface.py`
- Defines common interface for all LLM backends
- Methods: `chat_completion()`, `generate_response()`, `stream_generate_response()`

**Implementations**:

1. **AnythingLLMInterface** (`anythingllm_interface.py`)
   - HTTP client for AnythingLLM API
   - Supports streaming via Server-Sent Events (SSE)
   - Handles workspace management
   - Configuration: base_url, workspace_slug, API key

2. **NpuLLMInterface** (`npu_llm_interface.py`)
   - Direct QAI Hub models (Phi-3.5-Mini, etc.)
   - NPU inference via QNN runtime
   - Auto-discovers .qnn files in models directory
   - Optimized for on-device execution

3. **LocalLLMInterface** (`local_llm_interface.py`)
   - Wraps llama-cpp-python for CPU inference
   - Loads GGUF quantized models
   - Fallback when NPU unavailable
   - Pure Python implementation

### 4. Planning System (`planner_ollama_interface.py`)

**Purpose**: Generate task decomposition plans (DPPM format)

**PlannerOllamaInterface**:
- Calls Ollama API with structured prompt
- Validates plan JSON schema
- Returns structured plan data

**Plan Structure**:
```json
{
  "subtasks": [
    {
      "id": "S1",
      "title": "Task title",
      "objective": "What to do",
      "agent_key": "agent_name",
      "engine": "anythingllm",
      "parallel_group": 1,
      "depends_on": []
    }
  ],
  "merge": {
    "strategy": "How to combine results",
    "steps": [...]
  }
}
```

**Planning Flow**:
1. User enters goal with `/plan <goal>`
2. Planner decomposes into subtasks
3. Validation checks plan structure
4. User confirms before execution
5. Plan saved to `memory/plans/`

### 5. Execution System (`execution_dispatcher.py`)

**Purpose**: Execute planned subtasks with fault tolerance

**ExecutionDispatcher**:
- Loads plan from JSON
- Executes each subtask via LLM backends
- Manages retry logic (2 attempts by default)
- Tracks task status: pending → running → completed/failed
- Saves results to memory

**Execution Pipeline**:
```
For each subtask:
  1. Try Backend 1 (AnythingLLM)
  2. On failure → Try Backend 2 (QNN)
  3. On failure → Try Backend 3 (CPU)
  4. On all failure → Abort plan with error
  5. Save result to memory
  6. Update plan JSON with result path
```

**Retry Strategy**:
- `retry_attempts`: Number of retries (default 2)
- `retry_delay`: Wait between retries (default 5s)
- Exponential backoff for network errors

### 6. Memory System (`memory_system.py`)

**Purpose**: Persistent conversation and plan storage

**Structure**:
```
memory/
├── plans/                    # Saved execution plans (JSON)
│   └── 20250101-120000_goal-name.json
├── code_helfer/              # Agent memory (categories)
│   ├── agent1_20250101-120000.txt
│   └── agent1_20250101-120001.txt
├── projektmanager/
│   └── ...
└── general/                  # Default category
    └── ...
```

**File Format** (text-based conversations):
```
---
Agent: Code Helper
AgentKey: code_helfer
Workspace: main
Timestamp: 2025-01-01 12:00:00
Tags: python, debugging
---
System Prompt:
[system instructions]
---
User:
[user question]
---
SelfAI:
[ai response]
```

**Key Features**:
- Automatic category assignment
- Tag extraction from content
- Context filtering (relevance-based)
- Plan serialization to JSON

### 7. Context Filtering (`context_filter.py`)

**Purpose**: Smart retrieval of relevant conversation history

**Algorithms**:
1. **Task Classification**: Categorize user input (coding, planning, etc.)
2. **Relevance Scoring**: Calculate similarity to past conversations
3. **Context Selection**: Retrieve top N most relevant past interactions

**Integration**: Used by `load_relevant_context()` to populate chat history

### 8. Terminal UI (`ui/terminal_ui.py`)

**Purpose**: Rich terminal interface with animations

**Features**:
- ASCII banner and spinners
- Progress bars for long operations
- Color-coded status messages (green=success, yellow=warning, red=error)
- Typing animation for AI responses
- Stream prefix labels (showing which backend)
- Plan visualization with tree structure
- Interactive menu selection

**Status Levels**:
- `"success"` - Green ✓
- `"info"` - Blue ⓘ
- `"warning"` - Yellow ⚠
- `"error"` - Red ✗

---

## Entry Points & Execution Flow

### Main Entry Point: `selfai/selfai.py` (Recommended)

**Complete 3-phase pipeline**:

```python
python /path/to/selfai/selfai.py
```

**Flow**:
1. Initialize configuration, agents, memory, UI
2. Load LLM backends in priority order (AnythingLLM → QNN → CPU)
3. Load optional planner providers (Ollama)
4. Load optional merge providers (Ollama)
5. Enter interactive loop:
   - `/plan <goal>` → Planning phase
   - Normal message → Chat (execution phase)
   - `/memory` → Manage memory
   - `/switch <agent>` → Switch agents
   - `quit` → Exit

**Key Commands**:
- `/plan <goal>` - Create and execute task decomposition plan
- `/planner list` - List available planner backends
- `/planner use <name>` - Switch planner provider
- `/memory` - List memory categories
- `/memory clear <category>` - Clear memory
- `/switch <agent_name|number>` - Switch active agent
- `quit` - Exit program

### Alternative Entry Point: `main.py`

**Simple agent initialization**:

```python
python main.py
```

**Flow**:
1. Initialize Agent with local-ollama provider
2. Simple chat loop without planning/merge
3. Tool-based execution (via `smolagents`)

**Use Case**: Basic testing without complex infrastructure

### Alternative Entry Point: `llm_chat.py`

**Direct QNN/NPU chat**:

```python
python llm_chat.py
```

**Features**:
- Direct QAI Hub model loading
- Phi-3.5-Mini on Snapdragon X Elite NPU
- Simple interactive chat
- No configuration needed

---

## Component Interaction Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                        selfai.py (Main Loop)                     │
└────────────────────────┬─────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    ┌────────┐      ┌─────────┐     ┌──────────┐
    │Planning│      │Execution│     │  Merge   │
    │ Phase  │      │  Phase  │     │  Phase   │
    └────┬───┘      └────┬────┘     └────┬─────┘
         │               │               │
         ▼               ▼               ▼
   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
   │PlannerOllama │ │ExecutionDisp │ │MergeOllama   │
   │Interface     │ │atcher        │ │Interface     │
   └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
          │                │                │
          └────────────────┼────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
    ┌────────────┐   ┌────────────┐   ┌────────────┐
    │AnythingLLM │   │NpuLLM      │   │LocalLLM    │
    │Interface   │   │Interface   │   │Interface   │
    └────────────┘   └────────────┘   └────────────┘
        ▲                  ▲                  ▲
        │                  │                  │
    ┌───┴──────────────────┼──────────────────┴───┐
    │                      │                      │
    │ ┌────────────────────┘                      │
    │ │ Automatic Fallback in Priority Order      │
    │ │                                           │
    ▼ ▼                                           ▼
┌──────────┐  ┌────────────┐  ┌──────────────┐
│AnythingLLM│ │ QNN Models │  │ GGUF Models  │
│Server     │ │ (.qnn)     │  │ (CPU)        │
│(NPU)      │ │ (NPU)      │  │              │
└──────────┘  └────────────┘  └──────────────┘


┌─────────────────────────────────────────────────┐
│            Supporting Systems                   │
├─────────────────────────────────────────────────┤
│  AgentManager → Agent instances & switching    │
│  MemorySystem → Conversation & plan persistence│
│  ConfigLoader → Centralized configuration      │
│  TerminalUI   → Rich terminal interface        │
│  ContextFilter→ Smart context retrieval        │
└─────────────────────────────────────────────────┘
```

---

## Dependencies

### Core Dependencies (`requirements-core.txt`)

```
PyYAML              # Config file parsing
python-dotenv       # Environment variable loading
openai              # OpenAI API compatibility
llama-cpp-python    # CPU model inference (GGUF)
numpy               # Numerical computing
pyarrow             # Data serialization
tabulate            # Table formatting
smmap               # Fast file mapping
psutil              # System monitoring
qai-hub-models      # Qualcomm AI Hub models
smolagents          # Agent toolkit
```

### NPU Dependencies (`requirements-npu.txt`)

```
httpx==0.28.1       # HTTP client for API calls
qai_hub_models      # QNN model support
```

### System Requirements

**Hardware**:
- Windows on ARM (Snapdragon X Elite or compatible)
- Minimum 8GB RAM for CPU inference
- 16GB+ recommended for NPU optimization

**Software**:
- Python 3.12 (ARM64 build)
- AnythingLLM Desktop (ARM64) - Optional for NPU backend
- Ollama - Optional for planning/merge phases

---

## Setup Instructions

### 1. Initial Setup

```bash
# Clone repository
git clone <repository-url>
cd AI_NPU_AGENT_Projekt

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows CMD: .\.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configuration

```bash
# Copy and configure
cp config.yaml.template config.yaml
cp .env.example .env

# Edit config.yaml with your settings
# Edit .env with your AnythingLLM API key
```

### 3. Prepare Models

```bash
# Create models directory
mkdir -p models

# Download GGUF model for CPU fallback
# Place in models/ directory
# e.g., Phi-3-mini-4k-instruct.Q4_K_M.gguf
```

### 4. Set Up Agents

```bash
# Create agents directory
mkdir -p agents

# Create agent directories with:
# agents/agent_key/
#   ├── system_prompt.md
#   ├── memory_categories.txt
#   ├── workspace_slug.txt
#   └── description.txt
```

### 5. Optional: Start Ollama (for planning/merge)

```bash
# If using Ollama planner/merge
ollama serve

# In another terminal, pull models
ollama pull gemma3:1b
ollama pull gemma3:3b
```

### 6. Optional: Start AnythingLLM

```bash
# If using AnythingLLM for primary inference
# Launch AnythingLLM Desktop and configure workspace
```

---

## Common Workflows

### Workflow 1: Simple Chat

```
python selfai/selfai.py
> You: What is Python?
AI: [Response from available backend]
```

### Workflow 2: Task Decomposition with Planning

```
python selfai/selfai.py
> You: /plan Create a Python web crawler for news sites
[Planner decomposes into subtasks]
[System executes each subtask]
[Merge synthesizes final solution]
```

### Workflow 3: Agent Switching

```
python selfai/selfai.py
> You: /switch projektmanager
Switched to: Project Manager
> You: Analyze the project requirements
AI: [Response from project manager agent]
```

### Workflow 4: Memory Management

```
python selfai/selfai.py
> You: /memory
Aktive Memory-Kategorien:
- code_helfer
- projektmanager

> You: /memory clear code_helfer
Memory 'code_helfer' komplett geleert (15 Einträge).
```

---

## How to Extend

### Adding a New Agent

1. Create directory:
   ```
   agents/my_agent/
   ├── system_prompt.md       (Agent personality)
   ├── memory_categories.txt  (One per line)
   ├── workspace_slug.txt     (AnythingLLM workspace)
   └── description.txt        (What agent does)
   ```

2. Reference in config.yaml:
   ```yaml
   agent_config:
     default_agent: "my_agent"
   ```

### Adding a New Tool

1. Create in `selfai/tools/`:
   ```python
   class MyTool:
       @property
       def name(self):
           return "my_tool"
       
       @property
       def description(self):
           return "Tool description"
       
       @property
       def inputs(self):
           return {
               "param1": {"description": "..."}
           }
       
       def run(self, param1: str) -> str:
           # Implementation
           return result
   ```

2. Register in `selfai/tools/tool_registry.py`:
   ```python
   from selfai.tools.my_tool import MyTool
   # Add to registry
   ```

### Adding New LLM Backend

1. Create new interface in `selfai/core/`:
   ```python
   class MyLLMInterface:
       def generate_response(self, ...): ...
       def stream_generate_response(self, ...): ...
   ```

2. Instantiate in `selfai/selfai.py`:
   ```python
   interface, label = _load_my_llm(models_root, ui)
   execution_backends.append({
       "interface": interface,
       "label": label,
       "name": "my_backend"
   })
   ```

---

## Troubleshooting

### Issue: "API_KEY is not set"

**Solution**: 
- Copy `.env.example` to `.env`
- Add your AnythingLLM API key
- Ensure `config.yaml` is created from template

### Issue: "AnythingLLM not available"

**Solution**:
- Verify AnythingLLM server running on configured host:port
- Check `npu_provider.base_url` in config.yaml
- System will automatically fall back to QNN or CPU

### Issue: CPU inference very slow

**Solution**:
- Reduce `max_output_tokens` in config
- Use quantized models (Q4_K_M)
- Reduce `n_ctx` (context window)
- Consider using AnythingLLM + NPU for acceleration

### Issue: Memory growing unbounded

**Solution**:
- Use `/memory clear <category>` to manage
- Or: `/memory clear <category> 5` to keep only last 5
- Memory is organized by agent memory_categories

### Issue: Planner not working

**Solution**:
- Ensure Ollama running: `ollama serve`
- Set `planner.enabled: true` in config.yaml
- Verify Ollama models installed: `ollama pull gemma3:1b`
- Check `planner.providers[0].base_url` points to Ollama

---

## Performance Considerations

### Streaming vs. Blocking

- **Streaming** (default): Better UX, lower latency perception
  - Enable: `system.streaming_enabled: true`
  - Supported by: AnythingLLM, some Ollama versions

- **Blocking**: Simple, predictable latency
  - Use if streaming unavailable
  - System falls back automatically

### Backend Selection

| Backend | Speed | Quality | Hardware | Notes |
|---------|-------|---------|----------|-------|
| AnythingLLM (NPU) | Fast | High | Snapdragon X Elite | Recommended primary |
| QNN | Very Fast | High | Snapdragon X Elite | Direct NPU access |
| CPU (GGUF) | Slow | Medium | Any | Fallback guarantee |

### Token Limits

- Planner `max_tokens`: 768 (plan generation)
- Merge `max_tokens`: 1536 (result synthesis)
- Chat `max_output_tokens`: 512 (regular response)
- Increase for longer responses, decrease for speed

---

## Technical Details

### DPPM (Distributed Planning Problem Model) Format

The planner generates plans in DPPM format:

```json
{
  "subtasks": [
    {
      "id": "S1",
      "title": "Analyze Requirements",
      "objective": "Understand what user needs",
      "agent_key": "analyst",
      "engine": "anythingllm",
      "parallel_group": 1,
      "depends_on": [],
      "result_path": "memory/plans/results/S1.txt"
    },
    {
      "id": "S2",
      "title": "Design Solution",
      "objective": "Create architecture",
      "agent_key": "architect",
      "engine": "anythingllm",
      "parallel_group": 2,
      "depends_on": ["S1"],
      "result_path": "memory/plans/results/S2.txt"
    }
  ],
  "merge": {
    "strategy": "Combine analysis and design",
    "steps": [
      {
        "title": "Synthesis",
        "description": "Unite results",
        "depends_on": ["S2"]
      }
    ]
  },
  "metadata": {
    "planner_provider": "local-ollama",
    "planner_model": "gemma3:1b",
    "goal": "Create a web application",
    "merge_agent": "projektmanager"
  }
}
```

### Streaming Protocol

AnythingLLM and Ollama use **Server-Sent Events (SSE)**:

```
event: message
data: {"content": "Hello"}

event: message
data: {"content": " world"}

event: end
data: {"done": true}
```

System automatically decodes and displays streaming chunks.

### Configuration Validation

1. **Type Checking**: Dataclass validation
2. **Required Fields**: Missing keys raise ValueError
3. **URL Validation**: Tested with health checks
4. **Model Existence**: GGUF files checked before use

---

## Development Notes

### Code Organization Principles

1. **Separation of Concerns**: Each module has single responsibility
   - `*_interface.py` → Backend communication
   - `*_system.py` → Persistent state
   - `execution_*` → Task orchestration
   - `ui/` → User interface

2. **Dependency Injection**: Core business logic independent of I/O
   - Interfaces passed as parameters
   - Easy to mock for testing
   - Flexible backend switching

3. **Graceful Degradation**: System continues with reduced capability
   - Missing optional features don't crash
   - Fallback mechanisms at each level
   - Clear status messages about limitations

4. **Configuration-Driven**: Behavior changes without code modification
   - All settings in config.yaml
   - Environment variable interpolation
   - Validated at startup

### Key Design Patterns

1. **Strategy Pattern**: Multiple interchangeable LLM backends
2. **Chain of Responsibility**: Backend fallback chain
3. **Observer Pattern**: UI status callbacks
4. **Repository Pattern**: Memory system abstraction
5. **Factory Pattern**: Agent and tool instantiation

---

## Security Considerations

### Secrets Management

- **Never commit** `.env` file or API keys
- Use `.env.example` as template
- Load secrets via `python-dotenv`
- Validate all API keys at startup

### Input Validation

- User prompts passed through context filters
- Plan validation before execution
- Tool arguments checked before execution
- Configuration values type-checked

### File Operations

- All file I/O uses `pathlib` for safety
- Paths resolved relative to project root
- No arbitrary shell execution (unless explicit)
- Memory files stored in secure locations

---

## Future Enhancements

Potential improvements:

1. **Parallel Subtask Execution**: Execute independent tasks concurrently
2. **Custom Tools**: User-defined tool integration
3. **Multi-Model Ensemble**: Combine outputs from multiple backends
4. **RAG Integration**: Vector database for document retrieval
5. **Web UI**: Browser-based interface instead of CLI
6. **Distributed Execution**: Execute subtasks on remote machines
7. **Audio I/O**: Voice input/output support
8. **Plugin System**: Dynamic backend/tool loading

---

## References

### Related Files

- **Configuration**: `config.yaml.template`, `config_extended.yaml`
- **Models**: Download from Hugging Face, Ollama, QAI Hub
- **Documentation**: `README.md`, `UI_GUIDE.md`
- **Examples**: Check `Learings_aus_Problemen/` directory

### External Resources

- [AnythingLLM Documentation](https://anythingllm.com/)
- [Ollama Models](https://ollama.ai/library)
- [QAI Hub Models](https://huggingface.co/collections/qualcomm-ai-research/qai-hub-models)
- [Llama.cpp](https://github.com/ggerganov/llama.cpp)
- [GGUF Format](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)

---

## Version History

- **Current**: Multi-backend inference with planning/merge phases
- **Previous**: Simple chatbot with NPU/CPU fallback

---

**Last Updated**: January 2025
**Maintained By**: AI NPU Agent Project Team
**License**: [Check LICENSE file]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smlfg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
