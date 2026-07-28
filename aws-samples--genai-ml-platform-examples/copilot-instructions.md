## genai-ml-platform-examples

> This project demonstrates how to trace and evaluate LangChain LangGraph LLM agents using Amazon SageMaker MLflow. It features a ReAct-type financial assistant agent that integrates with MLflow for enhanced experimentation, management, observability, and evaluation.

# SageMaker MLflow LangGraph Agent Project

This project demonstrates how to trace and evaluate LangChain LangGraph LLM agents using Amazon SageMaker MLflow. It features a ReAct-type financial assistant agent that integrates with MLflow for enhanced experimentation, management, observability, and evaluation.

The core functionality includes agent creation (`graph.py`), evaluation utilities (`utils.py`), and MLflow integration for comprehensive agent lifecycle management.

## Build & Commands

- Install dependencies: `uv pip sync pyproject.toml`
- Create virtual environment: `uv venv`
- Run agent evaluation: `uv run run.py`
- Alternative with pip: `python run.py`
- Run Jupyter notebooks: `jupyter notebook`
- Activate environment: `source .venv/bin/activate`

### Development Environment

- Python version: >=3.13
- MLflow tracking server: Amazon SageMaker MLflow
- AWS Bedrock model: Claude 3.5 Haiku (inference profile)
- Virtual environment: `.venv/`
- Configuration: `.env` file (copy from `.env_sample`)

## Code Style

- Python: Type hints with modern Python features (>=3.13)
- Use descriptive variable/function names following snake_case convention
- Import organization: Standard library, third-party, local imports
- Environment variables: Use python-dotenv for configuration management
- Error handling: Implement proper exception handling for AWS services
- Documentation: Use docstrings for functions and classes
- Line length: Follow PEP 8 guidelines (79-88 characters)
- Use f-strings for string formatting
- Prefer pathlib for file operations
- NEVER hardcode AWS credentials or sensitive data

## Testing

- Jupyter notebooks for interactive testing and evaluation
- MLflow evaluation metrics for agent performance assessment
- RAGAS integration for comprehensive LLM evaluation
- Golden dataset: `golden_questions_answer.jsonl` for evaluation
- Test notebooks: `evaluations.ipynb`, `additional_evaluations_with_ragas.ipynb`
- Manual testing through `sagemaker_mlflow_strands_agent.ipynb`

## Architecture

- Agent Framework: LangGraph for stateful, multi-actor applications
- LLM Provider: AWS Bedrock (Claude 3.5 Haiku)
- Tracking: Amazon SageMaker MLflow
- Evaluation: MLflow LLM evaluation + RAGAS metrics
- Tools: Custom financial data retrieval tools
- State Management: LangGraph StateGraph with message handling
- Configuration: Environment-based configuration management

## Security

- Use AWS IAM roles and policies for service access
- Store sensitive configuration in environment variables
- Never commit `.env` files or AWS credentials to repository
- Use AWS SDK credential chain for authentication
- Validate all user inputs in agent tools
- Follow AWS security best practices for Bedrock and SageMaker
- Use least privilege principle for AWS permissions
- Regularly update dependencies for security patches

## Environment Setup

When setting up the project, configure these environment variables in `.env`:

1. `PROJECT`: Your project identifier
2. `MLFLOW_URI_SMAI`: SageMaker MLflow tracking server ARN
3. `MLFLOW_EXPERIMENT_ID`: MLflow experiment identifier
4. `VERSION`: Project version
5. `MODELID`: Bedrock model ID (e.g., Claude 3.5 Haiku inference profile)
6. `AWS_REGION`: AWS region for services

All configuration keys use UPPERCASE naming and MUST be documented in `.env_sample`.

## Agent Components

- **Graph Definition**: `graph.py` - LangGraph agent implementation
- **Tools**: `tools.py` - Custom tools for financial data retrieval
- **Utilities**: `utils.py` - MLflow integration and evaluation functions
- **Data**: `data.py` - Data handling and processing utilities
- **Prompts**: `mlflow_prompts.py` - MLflow prompt registry integration
- **Evaluation**: Jupyter notebooks for comprehensive agent testing

## MLflow Integration

- Experiment tracking with SageMaker MLflow
- Agent performance evaluation using MLflow LLM metrics
- Prompt registry for version control and management
- Trace logging for agent execution analysis
- Model registration and deployment capabilities
- Integration with RAGAS for extended evaluation metrics

## Troubleshooting

Common issues and solutions:
- ROCm library warnings can be safely ignored
- AttributeError with 'NoneType' rsmi_shut_down can be ignored
- Ensure AWS credentials are properly configured
- Verify SageMaker MLflow tracking server is accessible
- Check Bedrock model access permissions
- Validate environment variable configuration

---
> Source: [aws-samples/genai-ml-platform-examples](https://github.com/aws-samples/genai-ml-platform-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
