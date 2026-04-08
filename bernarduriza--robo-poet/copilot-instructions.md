## robo-poet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Robo-Poet** is an academic text generation framework that has undergone migration from TensorFlow to PyTorch. It's a modular system for training GPT-style transformer models on literary texts and generating coherent poetry/prose.

### Key Technologies
- **PyTorch** (primary deep learning framework)
- **Python 3.12+**
- **CUDA support** for GPU acceleration (NVIDIA RTX 2000 Ada target)
- **WSL2 + Windows 11** development environment

## Architecture Overview

### Hybrid Architecture Structure
The project contains three main implementations:

1. **Django Web Framework** (Primary Interface):
   - Full-stack web application with real-time capabilities
   - Django Channels for WebSocket communication
   - Interactive dashboard with live training metrics
   - Training session management and visualization

2. **Legacy Modular System** (`src/` directory):
   - Domain-driven design with clean architecture patterns
   - Enterprise-style structure with repositories, services, entities
   - Orchestrator-based workflow (`src/orchestrator.py`)
   - Headless mode for Django integration

3. **PyTorch Implementation** (`src/legacy/robo-poet-pytorch/` directory):
   - Simplified, research-focused structure (archived)
   - Direct CLI interface via `main.py`
   - MinGPT-style transformer architecture

### Core Components

#### Main Entry Points
- `manage.py` - Django web application server
- `robo_poet.py` - CLI orchestrator with Django integration
- `src/legacy/robo-poet-pytorch/main.py` - Legacy PyTorch CLI interface (archived)

#### Model Architecture (`src/models/gpt_pytorch.py`)
- MinGPT-inspired transformer with 6 layers, 8 heads, 256 embedding dimensions
- Target: <10M parameters, validation loss <5.0
- Optimized for small literary datasets (Shakespeare, Alice in Wonderland)

#### Configuration System (`src/core/unified_config.py`)
- Hierarchical configuration with environment variable support
- GPU backend configuration (CUDA/ROCM/CPU/AUTO)
- Training hyperparameters and system settings

#### Data Processing
- `src/data/` - Dataset preprocessing and vocabulary building
- `robo-poet-pytorch/src/data/shakespeare_dataset.py` - PyTorch data loading
- Supports multiple text corpora with unified processing

#### Intelligence Layer (NEW)
- `src/intelligence/claude_integration.py` - Claude AI integration for smart training
- `src/interface/phase3_intelligent_cycle.py` - Intelligent training cycle interface
- Real-time dataset optimization using Claude API feedback
- Adaptive training with AI-guided improvements

## Common Development Commands

### Django Web Interface (Primary)
```bash
# Start Django development server
python manage.py runserver 8000

# Access web interface
# Navigate to: http://localhost:8000

# Run Django migrations
python manage.py makemigrations
python manage.py migrate

# Test Django integration
python test_training_integration.py
```

### CLI Training (Alternative)
```bash
# Legacy system training
python robo_poet.py

# Quick test training
python robo_poet.py --test quick

# Intelligent training cycle with Claude AI
python robo_poet.py
# Select option 3: 🧠 FASE 3: Ciclo Inteligente con Claude AI

# Headless mode for Django integration
python robo_poet.py --headless
```

### Claude AI Integration Setup
```bash
# Install Claude AI dependencies
python setup_claude_integration.py

# Manual installation
pip install -r requirements_claude.txt

# Configure API key
export CLAUDE_API_KEY=your_api_key_here
```

### Text Generation
```bash
# Web interface generation
# Use Django dashboard at http://localhost:8000

# CLI generation (legacy)
python robo_poet.py
# Select option 2: Phase 2 - Text Generation

# Legacy PyTorch generation (archived)
cd src/legacy/robo-poet-pytorch
python main.py generate --checkpoint checkpoints/best.pth --prompt "To be or not to be"
```

### Testing and Validation
```bash
# Module 2 comprehensive test suite
python src/testing/module2_test_suite.py

# Legacy test runner (deprecated)
python src/utils/run_module2_tests.py --quick
```

### Django Web Features
```bash
# Real-time training monitoring via WebSocket
# Live GPU metrics and training progress
# Interactive training session management
# Chart.js visualization of loss curves
# Training history and session tracking

# WebSocket endpoint
ws://localhost:8000/ws/training/global/

# REST API endpoints
http://localhost:8000/training/api/sessions/
http://localhost:8000/training/api/gpu-status/
```

## Development Guidelines

### Model Training Targets
- **Parameters**: Target <10M for academic demonstrations
- **Loss**: Validation loss <5.0 for coherent generation
- **Context**: 128-token context window (configurable)
- **Generation**: Coherent 200+ token outputs

### GPU Optimization
- CUDA memory optimization with `max_split_size_mb:512`
- Mixed precision training support
- RTX 2000 Ada specific optimizations
- Fallback to CPU if GPU unavailable

### Data Processing
- Text preprocessing in `src/data/dataset_preprocessor.py`
- Character and word-level vocabularies supported
- Multi-corpus processing with metadata tracking
- Processed data stored in `data/processed/`

### Testing Strategy
- Academic validation through `Module2TestSuite`
- Quick tests (~5 minutes) vs full validation (~20 minutes)
- Gradient flow analysis and loss landscape evaluation
- Automated reporting with timestamped outputs

### Intelligent Training (NEW)
- **Claude AI Integration**: Real-time dataset optimization during training
- **Adaptive Cycles**: Train → Evaluate → Consult AI → Improve → Repeat
- **Smart Suggestions**: AI-guided dataset improvements based on model performance
- **Cost Effective**: Uses Claude Haiku model (~$0.25 per full cycle)
- **Fallback Support**: Works without AI when API unavailable

## Project Structure Context

### Django Web Framework (Primary)
```
robo_poet_web/      # Django project settings
├── settings.py     # Main configuration
├── urls.py         # URL routing
├── asgi.py         # WebSocket configuration
└── wsgi.py         # WSGI configuration

dashboard/          # Main dashboard app
├── views.py        # Dashboard views
├── models.py       # Dashboard models
└── migrations/     # Database migrations

training/           # Training management app
├── models.py       # TrainingSession, TrainingMetric
├── views.py        # Training API endpoints
├── consumers.py    # WebSocket consumers
├── routing.py      # WebSocket routing
└── urls.py         # REST API URLs

templates/          # Web templates
├── base/           # Base templates
└── dashboard/      # Dashboard templates
```

### Core AI System (`src/`)
```
src/
├── orchestrator.py         # Main orchestrator with Django integration
├── intelligence/           # Claude AI integration
│   └── claude_integration.py
├── infrastructure/         # External services
│   └── django_integration.py
├── interface/              # Phase interfaces
│   ├── phase1_training.py
│   ├── phase2_generation.py
│   └── phase3_intelligent_cycle.py
├── models/                 # Neural network implementations
├── data/                   # Data processing pipelines
└── utils/                  # Helper utilities
```

### Legacy PyTorch System (`src/legacy/robo-poet-pytorch/`)
```
src/legacy/robo-poet-pytorch/ (Archived)
├── src/
│   ├── models/     # GPT model implementation
│   ├── training/   # Training loops and optimization
│   ├── generation/ # Text generation utilities
│   └── utils/      # Vocabulary and preprocessing
└── main.py         # Legacy CLI interface
```

## Important Configuration

### Environment Setup
- Python 3.12+ required
- PyTorch with CUDA support recommended
- WSL2 environment optimizations included
- Environment variables in `.env.example`

### Line Endings
**CRITICAL**: All files must use CRLF line endings (Windows)
- Never convert to LF - breaks code consistency
- Verify `git config core.autocrlf true` and `git config core.eol crlf`

### Git Authorship
- All commits authored by "Bernard Uriza Orozco"
- No AI attribution in commit messages
- Conventional commit format: `feat:`, `fix:`, `refactor:`
- No emojis in commit messages

## Performance and Monitoring

### Key Metrics
- Training loss curves and validation tracking
- Gradient flow analysis for training stability
- Memory usage optimization for RTX 2000 Ada
- Generation quality through perplexity and human evaluation

### Debugging Tools
- Structured logging with timestamped outputs
- Hospital/surgery system for model debugging (`src/hospital/`)
- Gradient analysis utilities
- Loss landscape visualization

## Django Web Integration

### Real-time WebSocket Communication
- **WebSocket URL**: `ws://localhost:8000/ws/training/global/`
- **Real-time Metrics**: Live training loss, GPU usage, perplexity
- **Bidirectional Communication**: Web → CLI control commands
- **Session Tracking**: Database persistence of training sessions

### Django Models
```python
# TrainingSession - Tracks complete training runs
class TrainingSession(models.Model):
    name = models.CharField(max_length=200)
    status = models.CharField(choices=STATUS_CHOICES)
    current_epoch = models.IntegerField()
    current_loss = models.FloatField()
    claude_enabled = models.BooleanField()
    process_id = models.IntegerField()  # robo_poet.py PID

# TrainingMetric - Individual metric points
class TrainingMetric(models.Model):
    session = models.ForeignKey(TrainingSession)
    epoch = models.IntegerField()
    train_loss = models.FloatField()
    gpu_memory_used = models.FloatField()
```

### Headless Mode Integration
- **Environment Variables**: `DJANGO_RUN=true`, `TRAINING_SESSION_ID=123`
- **WebSocket Reporter**: `src/infrastructure/django_integration.py`
- **CLI Integration**: `robo_poet.py --headless` for web execution
- **Process Monitoring**: Subprocess control via WebSocket commands

## Notes for Development

- **Primary Interface**: Django web dashboard at `http://localhost:8000`
- **CLI Alternative**: `python robo_poet.py` for direct terminal access
- **Real-time Updates**: WebSocket integration for live metrics
- **Database Tracking**: All training sessions persisted in SQLite/PostgreSQL
- **Multi-phase Support**: Phase 1 (Training), Phase 2 (Generation), Phase 3 (Intelligent AI)
- **Claude AI Integration**: Intelligent training cycles with cost tracking
- **GPU Optimization**: RTX 2000 Ada specific memory management
- **Testing Suite**: Comprehensive integration testing via `test_training_integration.py`

When working with this codebase, prefer the **Django web interface** for user interactions and training management. The CLI system (`src/orchestrator.py`) now supports headless mode for seamless web integration. The legacy PyTorch implementation (`src/legacy/robo-poet-pytorch/`) is archived but maintained for reference.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BernardUriza)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BernardUriza)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
