## ingenier-a-de-soluciones-con-inteligencia-artificial

> This file provides guidance to GEMINI when working with code in this repository.

# GEMINI.md

This file provides guidance to GEMINI when working with code in this repository.

## Project Overview

This is an educational repository for "Ingeniería de Soluciones con Inteligencia Artificial" (AI Solutions Engineering) course. The project follows a comprehensive three-phase learning structure covering generative AI fundamentals, intelligent agents development, and production deployment considerations.

## Architecture & Structure

### Learning Experiences (RA)
- **RA1 - Fundamentals of Generative AI and Prompt Engineering**
  - IL1.1: Introduction to LLMs and API connections
  - IL1.2: Prompt engineering techniques (zero-shot, few-shot, chain-of-thought)
  - IL1.3: RAG (Retrieval-Augmented Generation) infrastructure design
  - IL1.4: Evaluation and optimization of LLMs
  
- **RA2 - Development of Intelligent Agents with LLM**
  - IL2.1: Agent architecture and frameworks (LangChain, CrewAI)
  - IL2.2: Memory systems and external tool integration (Model Context Protocol)
  - IL2.3: Planning and orchestration strategies
  - IL2.4: Technical documentation and architecture design
  
- **RA3 - Observability, Security and Ethics in AI Agents**
  - IL3.1: Observability tools and performance metrics
  - IL3.2: Traceability analysis and log processing
  - IL3.3: Security protocols and ethical considerations
  - IL3.4: Scalability and sustainability strategies

### Assessment Structure
- **Formative Evaluations**: Quizzes (8 questions each) on theoretical concepts
- **Partial Evaluations**: Practical projects with presentations/deliverables
- **Final Transversal Evaluation**: Comprehensive project (40% weighting) covering all learning outcomes

### Code Organization
- Each RA contains introductory materials and specific learning modules (IL)
- Jupyter notebooks (.ipynb) contain practical implementations
- Markdown files (.md) provide conceptual guidance and requirements
- Projects are developed in pairs with individual presentations

## Development Environment

### Required Environment Variables
The notebooks require these environment variables for API access:
- `GITHUB_BASE_URL`: Base URL for GitHub Models API
- `GITHUB_TOKEN`: GitHub personal access token for model access
- `OPENAI_BASE_URL`: Alternative base URL for OpenAI-compatible APIs

### Key Dependencies
Based on notebook imports and course materials:
- `openai`: For direct OpenAI API calls
- `langchain` and `langchain-openai`: For LangChain framework integration
- Agent frameworks: LangChain, CrewAI for multi-agent systems
- Observability tools: LangSmith, Langfuse, Arize for monitoring
- Standard Python libraries: `os`, `pandas`, `requests`

### Development Workflow
- Environment setup with Python and Jupyter Notebook
- API configuration with authentication keys
- Progressive implementation from basic API calls to complex agent systems
- Documentation requirements include README.md files for agent implementations

## Common Patterns

### API Integration
- Uses OpenAI-compatible APIs through environment variables
- Supports both direct OpenAI client and LangChain abstractions
- Temperature and token limits are commonly configured for different use cases

### Agent Development
- Function calling integration with external tools
- Memory systems implementation (short-term and long-term)
- Planning algorithms for multi-stage task execution
- Error handling and recovery mechanisms

### Educational Progression
- Conceptual introduction (markdown) → practical implementation (notebook)
- Individual skill building → collaborative project development
- Progressive complexity from basic prompts to production-ready agents
- Strong emphasis on organizational/business context applications

### Project Deliverables
- Technical documentation with architecture diagrams
- Performance metrics and observability dashboards
- Security and ethical considerations documentation
- Scalability and deployment strategies

---
> Source: [davila7/Ingenier-a-de-Soluciones-con-Inteligencia-Artificial](https://github.com/davila7/Ingenier-a-de-Soluciones-con-Inteligencia-Artificial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
