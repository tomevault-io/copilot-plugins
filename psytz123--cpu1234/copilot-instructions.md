## cpu1234

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Codebase Overview

This is an AI-enhanced CPU performance optimization suite for Intel i7-1240P systems with 32GB DDR4-5200 memory. The codebase implements hardware-specific optimizations, cloud GPU integration, machine learning-based workload prediction, and advanced AI coding assistance with Model Context Protocol (MCP) integration for enhanced context windows.

## Critical Commands

### Running the Application
```bash
# Main launcher with all tools
python performance_tools_launcher.py

# Individual components
python src/optimization/ai_coding_optimizer.py          # AI workload optimization
python src/dashboard/unified_performance_dashboard.py   # GUI dashboard
python src/dashboard/realtime_dashboard_server.py       # WebSocket server (port 8765)

# New AI coding optimization components
python src/optimization/profile_manager.py              # Profile management system
python src/optimization/mcp_manager.py                  # MCP server management
python src/optimization/mcp_context_server.py           # Custom MCP context server
```

### Testing and Validation
```bash
# Test individual modules
python src/optimization/hybrid_cpu_optimizer.py         # Test hybrid CPU detection
python src/monitoring/memory_bandwidth_monitor.py       # Test memory bandwidth
python src/monitoring/gpu_acceleration_monitor.py       # Test GPU/QuickSync
python src/analysis/ml_workload_predictor.py           # Test ML predictions

# Verify async operations
python src/core/async_operations.py                     # Test async framework
```

### Configuration and Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Main config in cpu_monitor_config.json
# RunPod API key in .env (RUNPOD_API_KEY)
# ML models stored in ml_models/ directory

# AI Coding Optimization Setup
# Master config: ai_coding_master_config.json
# MCP servers config: mcp_servers_config.json  
# IDE optimizations: ide_optimizations_config.json
# Profiles: profiles_config.json
# Complete guide: AI_CODING_OPTIMIZATION_GUIDE.md

# Hybrid development setup with RunPod
# SSH connection: ssh awwtod4vlp17uz-644117b2@ssh.runpod.io -i ~/.ssh/id_ed25519
# Use Cursor Remote-SSH for direct cloud GPU development
```

## Architecture Overview

### Core System Architecture
The codebase follows a modular architecture with specialized components:

1. **Shared Utilities Layer** (`shared_utils.py`)
   - Central metrics collection (SystemMetrics, ProcessMonitor)
   - AI workload detection (AIWorkloadDetector)
   - CPU governor management (CPUGovernorManager)
   - Thermal monitoring (ThermalThrottlingDetector)
   - All other modules depend on this foundation

2. **Hardware Optimization Stack** (`src/optimization/`, `src/monitoring/`)
   - `src/optimization/hybrid_cpu_optimizer.py`: Intel Thread Director integration, P/E core management
   - `src/monitoring/memory_bandwidth_monitor.py`: DDR4-5200 bandwidth tracking, channel utilization
   - `src/monitoring/gpu_acceleration_monitor.py`: Intel QuickSync, GPU memory, AI framework GPU usage
   - `src/optimization/intel_xtu_integration.py`: Voltage control, power limits, thermal management

3. **AI/Cloud Integration Layer** (`src/cloud/`)
   - `src/cloud/runpod_gpu_integration.py`: Cloud GPU offloading, job queue management
   - `src/cloud/workload_distributor.py`: Local vs cloud decision engine
   - `src/cloud/cost_tracker.py`: Budget management, ROI calculations
   - `src/cloud/runpod_template_manager.py`: Docker templates for cloud execution

4. **Intelligence Layer** (`src/analysis/`, `src/optimization/`)
   - `src/analysis/ml_workload_predictor.py`: Scikit-learn models for workload forecasting
   - `src/optimization/ai_coding_optimizer.py`: Main optimization engine, profile management
   - `src/optimization/profile_manager.py`: Environment profile management with auto-switching
   - Pattern recognition and proactive optimization

5. **AI Coding Enhancement Layer** (`src/optimization/`)
   - `src/optimization/mcp_manager.py`: Model Context Protocol server orchestration
   - `src/optimization/mcp_context_server.py`: Custom MCP server for project context
   - Enhanced AI workload detection with expanded process patterns
   - IDE-specific optimizations and AI assistant tuning

6. **User Interface Layer** (`src/dashboard/`)
   - `src/dashboard/unified_performance_dashboard.py`: Tkinter GUI with matplotlib charts
   - `src/dashboard/realtime_dashboard_server.py`: WebSocket server with aiohttp
   - `performance_tools_launcher.py`: CLI menu system

### Key Design Patterns

**Event-Driven Architecture**: The system uses threading and async patterns throughout:
- Background monitoring threads in most modules
- AsyncEventEmitter in `src/core/async_operations.py` for non-blocking events
- WebSocket for real-time updates

**Factory Pattern**: Workload detection and optimization profiles:
- WorkloadType enums define categories
- Profile factories create optimized configurations
- Strategy pattern for different optimization approaches

**Data Pipeline**: Metrics flow through the system:
```
Hardware → SystemMetrics → Collectors → Analyzers → Optimizers → Actions
                              ↓
                         ML Predictor → Recommendations
```

### Critical Dependencies

**External Services**:
- RunPod API for cloud GPU (requires API key in .env)
- Intel XTU for Windows CPU optimization
- Docker for container monitoring

**Platform-Specific**:
- Windows: WMI for performance counters, registry for settings
- Linux: /proc, /sys filesystems, perf tools
- WSL2: Requires Windows 11 build 22000+

### Performance Considerations

1. **Caching Strategy**: Most modules cache metrics for 1-5 seconds to reduce overhead
2. **Thread Safety**: Monitoring threads use daemon mode and proper cleanup
3. **Database**: SQLite for historical data (system_metrics.db)
4. **Memory**: ML models kept in memory after training (~100MB)

### Common Development Patterns

When adding new features:
1. Extend shared_utils.py for new metric types
2. Use src/core/async_operations.py patterns for non-blocking code
3. Add to config.py for new configuration options
4. Register with performance_tools_launcher.py for CLI access

When debugging performance issues:
1. Check cpu_monitor.log for detailed traces
2. Use get_optimization_report() methods for module status
3. Monitor system_metrics.db for historical patterns

### API Keys and Sensitive Data

- RunPod API key must be in `.env` file as `RUNPOD_API_KEY`
- Never commit .env file
- Configuration in cpu_monitor_config.json is safe to commit

### Module Initialization Order

For proper initialization:
1. config.py loads first (app_config global)
2. shared_utils.py provides base classes
3. Hardware modules initialize platform-specific features
4. AI modules load models if available
5. UI modules start last

### WebSocket Real-time Updates

The src/dashboard/realtime_dashboard_server.py runs on port 8765 and provides:
- `/` - HTML dashboard
- `/ws` - WebSocket endpoint for metrics
- `/api/metrics` - REST endpoint for current metrics
- `/api/optimize` - Trigger optimization

### Machine Learning Models

ML models are stored in `ml_models/` directory:
- Auto-trains after 1000 samples collected
- Uses GradientBoostingRegressor for predictions
- Models: cpu_model.pkl, memory_model.pkl, gpu_model.pkl, thermal_model.pkl
- Retrain with: `predictor.train_models()` after data collection

### Hybrid Development Workflow

**Local System (Intel i7-1240P):**
- Primary development environment with optimized CPU performance
- Code editing, testing, and light AI workloads
- Git repository management and project coordination

**Cloud GPU (RunPod RTX 4090):**
- Heavy AI model training and inference
- Large dataset processing
- GPU-accelerated development via Cursor Remote-SSH

**Integration Points:**
- SSH config at `~/.ssh/config` for seamless connection
- Cursor Remote-SSH extension for cloud GPU development
- Automatic workload distribution between local CPU and cloud GPU
- Cost optimization with auto-shutdown capabilities (~$0.34/hour)

## AI Coding Optimization Features

### Model Context Protocol (MCP) Integration

The system now supports MCP servers for enhanced AI assistant context:

**MCP Server Management** (`src/optimization/mcp_manager.py`):
- Orchestrates up to 8 MCP servers simultaneously
- Smart load balancing and context routing
- Automatic failover and health monitoring
- Context caching and compression

**Available MCP Servers** (configured in `mcp_servers_config.json`):
- `filesystem`: Project file access
- `git`: Repository history and branch information  
- `database`: System metrics and performance data
- `web`: External documentation and research
- `custom_context`: Project-specific AI coding context
- `code_analysis`: Code structure analysis
- `performance_metrics`: Real-time system status
- `runpod_integration`: Cloud GPU status and management

**Context Window Expansion**:
- **2.5x context window expansion** through intelligent server orchestration
- Smart context selection based on query type and relevance
- Automatic context compression for token efficiency

### Environment Profile Management

**Profile System** (`src/optimization/profile_manager.py`, `profiles_config.json`):
- **6 optimization profiles**: development, ai_intensive, battery_saver, presentation, cloud_hybrid, debug_profile
- **Automatic profile switching** based on system conditions (battery, temperature, workload)
- **Intel i7-1240P specific optimizations**: P/E core affinity, memory channel optimization
- **IDE-specific settings** for different development scenarios

**Profile Features**:
- CPU governor management (performance/balanced/powersave)
- Memory allocation priorities and cache sizes
- MCP server activation policies
- AI assistant feature toggles
- Thermal management thresholds

### Enhanced AI Workload Detection

**Expanded Process Patterns** (in `shared_utils.py`):
- Modern AI coding tools: Cursor, Windsurf, Zed, Supermaven, Continue, Aider
- Inference engines: vLLM, TensorRT, LlamaCPP, ExLLaMA
- Vector databases: Pinecone, ChromaDB, Qdrant, Weaviate
- Model training frameworks: DeepSpeed, Accelerate, Ray, Lightning

**Detection Categories**:
- `language_models`: LLaMA, Mistral, CodeLLaMA, WizardCoder, StarCoder
- `inference_engines`: Production inference servers and optimized runtimes
- `vector_databases`: Semantic search and RAG systems
- `model_training`: Training and fine-tuning frameworks

### IDE Integration Optimizations

**Supported IDEs** (`ide_optimizations_config.json`):
- **Cursor**: High priority, performance core affinity, aggressive caching
- **VSCode**: Extension-specific optimizations for Copilot, TabNine, Codeium
- **PyCharm**: AI assistant optimization with 8GB heap, shared indexes
- **Vim/Neovim**: LSP optimization, AI plugin configuration
- **Windsurf**: Cascade AI integration, GPU acceleration
- **Zed**: Copilot integration, tree-sitter optimization

**Global Optimizations**:
- Language server CPU affinity and memory limits
- File watching exclusion patterns for performance
- Hardware acceleration and GPU compositing
- Parallel and incremental indexing

### Performance Enhancements

**Intel i7-1240P Specific Optimizations**:
- P-cores (0,2,4,6) for AI inference and compilation
- E-cores (8-15) for background tasks and language servers
- DDR4-5200 memory channel optimization
- Aggressive prefetching for large context windows

**Thermal Management**:
- Smart thermal prediction and throttling
- Temperature-based profile switching
- Fan curve optimization for sustained performance
- Thermal protection thresholds (78°C performance, 82°C balanced, 88°C emergency)

### Development Workflow Integration

**Configuration Files Hierarchy**:
1. `ai_coding_master_config.json` - Master configuration and feature flags
2. `cpu_monitor_config.json` - Core system optimization settings
3. `profiles_config.json` - Environment-specific profiles
4. `mcp_servers_config.json` - MCP server configuration
5. `ide_optimizations_config.json` - IDE-specific optimizations

**AI Coding Setup Commands**:
```bash
# Apply AI intensive profile for heavy development
python -c "from src.optimization.profile_manager import switch_to_profile; switch_to_profile('ai_intensive')"

# Start MCP servers for enhanced context
python -c "from src.optimization.mcp_manager import start_mcp_system; import asyncio; asyncio.run(start_mcp_system())"

# Get AI coding recommendations
python -c "from src.optimization.ai_coding_optimizer import AICodingOptimizer; optimizer = AICodingOptimizer(); print(optimizer.get_optimization_recommendations())"
```

This enhanced architecture provides a comprehensive AI coding platform with intelligent resource management, expanded context windows, and hardware-optimized performance for the Intel i7-1240P system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/psytz123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/psytz123)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
