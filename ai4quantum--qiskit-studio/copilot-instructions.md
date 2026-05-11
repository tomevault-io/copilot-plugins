## qiskit-studio

> This file provides guidance to AI development assistants when working with code in this repository.

# AGENTS.md

This file provides guidance to AI development assistants when working with code in this repository.

**Supported AI Assistants:**
- Bobshell (Cline)
- Claude Code
- GitHub Copilot
- Cursor AI
- Codeium
- Gemini CLI

## Project Overview

Qiskit Studio is a visual quantum computing development environment that bridges the gap between quantum theory and practical implementation. It's a Next.js web application that uses a node-based interface (ReactFlow) to build quantum circuits and generate Qiskit code, with AI-powered features to assist in the development process.

### Core Purpose
- Provide an intuitive visual interface for quantum computing development
- Generate Python/Qiskit code automatically from visual designs
- Offer AI-powered assistance for quantum programming
- Lower barriers for quantum computing newcomers while providing advanced features for experts

### Key Technologies
- **Frontend**: Next.js 14, TypeScript, ReactFlow, Tailwind CSS, shadcn/ui
- **Backend**: Python 3.9+, Maestro workflow engine
- **AI Integration**: Configurable AI service endpoints (Ollama, OpenAI, etc.)
- **Quantum SDK**: Qiskit for quantum computing operations

## Architecture

### Component Structure
1. **Node System**: The core of the application is a node-based graph built with ReactFlow
   - Each node represents a quantum computing component (circuit, gate, execution, etc.)
   - Nodes are connected to create quantum workflows
   - Node types are defined in `components/nodes/`

2. **AI Integration**:
   - Multi-agent architecture powered by Maestro
   - Three independent services:
     - Chat Agent (port 8000): Conversational AI with RAG
     - Code Generation Agent (port 8001): Quantum code generation
     - Code Execution Agent (port 8002): Safe Python/Qiskit execution
   - RAG (Retrieval-Augmented Generation) using Milvus vector database

3. **Code Generation**:
   - Automatic Python/Qiskit code generation from node configurations
   - AI-powered code improvement and optimization
   - Template-based code generation with AI enhancements

### Data Flow
```
User Interface → React Flow → Node System → Code Generation → AI Pipeline → Quantum Execution
     ↑                                                                              ↓
     ←─────────────────── Visualization ←─────────────────── Results Processing ←───
```

## Key Components

### Node Types
1. **Circuit Construction Nodes**: Circuit Library, Circuit, Gate nodes
2. **Execution Nodes**: Execution, Runtime nodes
3. **Optimization Nodes**: Transpiler, Transpiler AI Passes, Transpiler Pass nodes
4. **Analysis Nodes**: Quantum Info, Visualization nodes
5. **Specialized Nodes**: Chemistry Mapping, Python nodes

### Core Files and Directories
- `components/quantum-composer.tsx`: Central component orchestrating the ReactFlow canvas
- `components/nodes/`: Node definitions for different quantum operations
- `lib/api-service.ts`: Handles communication with AI backend
- `lib/code-generator.ts`: Logic for generating Python/Qiskit code
- `hooks/useAICodeGeneration.ts`: React hook for AI code generation

## Development Guidelines

### Environment Setup
1. **Prerequisites**:
   - Node.js v18+
   - Python 3.9+
   - uv (Python package manager)
   - Ollama (for local LLM support)

2. **Installation**:
   ```bash
   # Install frontend dependencies
   npm install
   
   # Configure environment
   cp .env.local.template .env.local
   # Edit .env.local with your configuration
   
   # Start frontend
   npm run dev
   ```

3. **AI Backend Setup**:
   - Download Ollama models (Ollama must be installed first - see Prerequisites):
   ```bash
   ollama pull granite3.3:8b
   ollama pull nomic-embed-text:latest
   ```
   
   - Start AI agents (in separate terminals):
   ```bash
   cd api/chat-agent && uv run maestro serve agents.yaml workflow.yaml
   cd api/codegen-agent && uv run maestro serve agents.yaml workflow.yaml --port 8001
   cd api/coderun-agent && uv run python agent.py --port 8002
   ```
   
   - Setup vector database (requires uv to be installed, recommend using a Python virtual environment):
   ```bash
   git clone https://github.com/AI4quantum/maestro-knowledge.git
   cd maestro-knowledge
   CUSTOM_EMBEDDING_URL=http://127.0.0.1:11434/v1 \
   CUSTOM_EMBEDDING_MODEL=nomic-embed-text \
   CUSTOM_EMBEDDING_VECTORSIZE=768 \
   CUSTOM_EMBEDDING_API_KEY=dummy \
   uv run ./start.sh
   
   # Populate database
   cd ../qiskit-studio/api/chat-agent/
   uv run python scripts/add-rag-docs-remote-embed.py
   ```

### Code Conventions

1. **Component Structure**:
   - React functional components with TypeScript
   - Props interfaces defined at the top of files
   - Custom hooks for shared logic

2. **State Management**:
   - React Context for global state
   - Local state for component-specific data
   - Custom hooks for complex state logic

3. **API Integration**:
   - All backend communication centralized in `lib/api-service.ts`
   - Error handling and response parsing standardized
   - Environment variables for API configuration

4. **Debugging**:
   - Console logs with prefixes (🎛️, 🚀, 🎣, 🔧) for tracing user interface interactions
   - Detailed error messages and state logging
   - Use browser developer console for troubleshooting
   - Enable debug mode by setting `NEXT_PUBLIC_DEBUG=true` in `.env.local` or using `localStorage.setItem('qiskit-studio-debug', 'true')` in the browser console

### Adding New Features

1. **Adding a New Node Type**:
   - Create a new React component in `components/nodes/MyNewNode.tsx`
   - Register it in the `nodeTypes` object in `components/quantum-composer.tsx`
   - Add it to the sidebar in `components/sidebar.tsx`
   - Implement node logic with `useAICodeGeneration` hook integration

2. **Modifying AI Integration**:
   - Update prompt templates in `lib/api-service.ts`
   - Add new API methods as needed
   - Update response parsing logic if required

3. **Extending Code Generation**:
   - Add new code templates in `lib/code-generator.ts`
   - Update parameter handling in `lib/api-service.ts`
   - Test with different node configurations

## Common Tasks

### Building and Running
```bash
# Development
npm run dev

# Production build
npm run build
npm run start

# Linting
npm run lint
```

### Testing
- **Manual Testing**: The primary method of testing is manual interaction through the web interface
- **Debug Tools**: Use browser developer console for troubleshooting
- **Automated Tests**: See `scripts/test/` for automated test scripts
- **Test Files**:
  - `lib/api-service.test.example.ts`: Example tests for the API service (not integrated into test runner)
  - `test/test_agent.py`: Python script for testing agent functionalities
- **Note**: The project does not currently have a comprehensive test suite (e.g., Jest or Vitest)

### Deployment
```bash
# Docker deployment
docker build -t qiskit-studio-ui .
docker run -p 3000:3000 --env-file .env.local qiskit-studio-ui:latest
```

## Troubleshooting

### Common Issues
1. **API Connection Problems**:
   - Check API URLs in `.env.local`
   - Ensure AI backend is running
   - Check network connectivity

2. **Node Interaction Issues**:
   - Check console logs with node-specific prefixes
   - Verify parameter update API is configured
   - Check API endpoints in `.env.local`

3. **Code Generation Failures**:
   - Check AI backend responses
   - Verify prompt templates
   - Check for syntax errors in generated code

## Project Goals

### Current Development
- Enhanced AI code generation with specialized quantum agents
- Extended quantum algorithm library
- Advanced visualization and analysis tools
- Collaborative workspace features

### Ideas for Future Enhancements
- Enhanced quantum hardware integration
- Multi-user collaborative editing
- Advanced quantum error correction tools
- Mobile-responsive interface
- Extended algorithm library

---
> Source: [AI4quantum/qiskit-studio](https://github.com/AI4quantum/qiskit-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
