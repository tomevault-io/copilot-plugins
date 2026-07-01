## banana-gen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Dependencies and Setup
```bash
# Install core dependencies
python example/install_dependencies.py

# Install Web UI dependencies
cd webui && pip install -r requirements.txt
```

### Running the Application

#### Web UI (Primary interface)
```bash
# Run from project root (recommended - auto-opens browser)
python start_webui.py

# Run from webui directory
cd webui && python start.py

# Alternative startup methods
cd webui && ./start.sh        # Linux/macOS
cd webui && start.bat         # Windows
```

#### CLI Examples
```bash
# Ultra-simple demo (recommended first run)
python example/demo_ultra_simple.py

# Simple task manager demo
python example/demo_simple_task_manager.py

# Nested workflow demo
python example/demo_nested_workflow.py
```

## Architecture Overview

### Core Framework Structure
This is a unified image generation management framework built around these key components:

- **UnifiedImageGenerator**: Central orchestrator managing Google AI API calls with concurrent processing
- **TaskManager**: High-level task automation with interactive monitoring and auto-fallback capabilities
- **AdvancedKeyManager**: Multi-tier API key pool with automatic failover and load balancing
- **PromptRegistry**: Structured prompt management with ID-based retrieval and input count categorization

### Module Organization
```
banana_gen/
├── core/           # Core framework definitions
├── keys/           # API key management (AdvancedKeyManager)
├── promptfiles/    # Prompt files directory (renamed from prompts/)
├── images/         # Image source abstraction (LocalImage, UrlImage, ImageFolder, etc.)
├── executor/       # Task execution (UnifiedImageGenerator, TaskManager)
├── output/         # Output path management and metadata embedding
├── logging/        # Structured logging with JSONL format
└── runner/         # Plan building and execution coordination
```

### Image Source System
The framework uses a flexible image source abstraction:

**Single Images**:
- `LocalImage`: Local file paths with fallback support
- `UrlImage`: Network images with fallback URLs
- `ImageData`: Direct image byte data
- `ImageGenerateTask`: Generated images from other tasks

**Image Collections**:
- `ImageFolder`: Local directories
- `ImageRecursionFolder`: Recursive directory scanning
- `ImageGenerateTasks`: Collections of generation tasks

All image sources support fallback paths for resilience.

### Task Management Patterns

#### Configuration-Based (Recommended)
```python
# Use input_configs for declarative setup
input_configs = [
    {
        "type": "folder",
        "main_path": "/path/to/images",
        "fallback_paths": ["/backup/path"]
    }
]

task_manager = TaskManager.create_with_auto_fallback(
    generator=generator,
    input_configs=input_configs,
    prompts=["prompt_id"],
    string_replace_list=[["old", "new"]]
)
```

#### Direct Object Creation
```python
# For advanced scenarios with custom logic
input_images = [
    ImageFolder("/path", fallback_paths=["/backup"]),
    LocalImage("/file.jpg")
]

task_manager = TaskManager(
    generator=generator,
    input_images=input_images,
    prompts=prompts
)
```

### Web UI Architecture
Flask-based web interface with multiple versions:
- `webui/app.py`: Original full-featured interface
- `webui/app_new.py`: Refactored clean interface
- `webui/app_refactored.py`: Modern Bootstrap 5 interface

The Web UI provides complete task management through browser interface at http://localhost:8888.

### Key Management
API keys are managed through text files in `banana_gen/keys/`:
- `api_keys_1.txt`: Priority 1 keys
- `api_keys_2.txt`: Priority 2 keys
- `api_keys_3.txt`: Priority 3 keys
- `api_keys_4.txt`: Priority 4 keys

The AdvancedKeyManager automatically handles key rotation, failure detection, and cooldown periods.

### Prompt System
Prompts are managed through:
- JSON registry files in `promptfiles/` directory (renamed from prompts/)
- Structured `Prompt` classes with metadata
- ID-based retrieval: `registry.get_prompts_by_ids(["p2_change_people"])`
- Categorization by input image count (0/1/2/3 input scenarios)

### Output Management
- Supports three path strategies: unified, separate paths, subdirectories
- Custom filename templates with placeholders
- PNG metadata embedding for generation parameters
- Automatic conflict resolution with unique naming

---
> Source: [dongshuyan/banana-gen](https://github.com/dongshuyan/banana-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
