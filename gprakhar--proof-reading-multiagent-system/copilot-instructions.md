## proof-reading-multiagent-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# IMPORTANT
- Never create a virtual env
- The virtual env folder is "/Users/gadappa/GenAI/genAI/proof_reading_multiagent_system/.virtenv"
- Always first activate the virtual env, before running a python command.
- Always first activate the virtual env, before installing dependencies.

## 1. Project Setup & Environment

### Project Root Directory
**Root Directory**: `/Users/gadappa/GenAI/genAI/proof_reading_multiagent_system`

This is the project root directory where all operations should be performed. Key characteristics:
- Contains the `.virtenv` virtual environment directory
- Contains the main `proof_reading_multiagent_system/` source code directory
- All Python commands, git operations, and development workflows should be run from this directory
- The virtual environment activation path is `source .virtenv/bin/activate` from this directory

### Memory Management (Development Environment)
**Target**: 8GB laptop development environment (production uses Google Cloud Run Functions)

#### Node.js Memory Configuration
```bash
# Add to ~/.zshrc for permanent configuration
export NODE_OPTIONS="--max-old-space-size=3072"  # 3GB heap limit for 8GB laptop
```

#### Memory Issue Troubleshooting
1. **Check Node.js memory limit**: Ensure `NODE_OPTIONS` is set to 3GB
2. **Monitor file sizes**: Keep development Excel files under 5MB when possible
3. **Restart when needed**: Don't hesitate to restart Claude Code if memory usage is high
4. **Browser management**: Close unnecessary tabs to free system memory

## 2. Architecture & Data Flow

### Project Overview
This is a **Proof Reading Multiagent System** built with CrewAI to automate document review and severity assessment for medical/scientific documents. The system uses two specialized agents to review document issues from Excel files and reassess their severity levels using configurable rulesets.

### CrewAI Architecture Pattern
- **Main Entry Point**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/main.py` - Contains execution functions (run, train, replay, test)
- **Crew Definition**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/crew.py` - Uses `@CrewBase` decorator to define agents and tasks
- **Agent Configuration**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/config/agents.yaml` - YAML-based agent definitions
- **Task Configuration**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/config/tasks.yaml` - YAML-based task definitions
- **Custom Tools**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/tools/` - Custom CrewAI tools extending `BaseTool`
- **Logging Infrastructure**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/utils/` - Comprehensive logging with Google Cloud integration

### Project Directory Structure
```
proof_reading_multiagent_system/
├── logs/                        # Root-level log files directory
│   ├── archive/                 # Archived log files by date
│   ├── proof_reading_system.log # Main system log file
│   └── session_*.log            # Session-specific log files
├── proof_reading_multiagent_system/  # Main project directory
│   ├── data/                    # Data files
│   │   ├── benchmarks/          # Performance benchmark data
│   │   ├── output/              # Generated output files
│   │   └── samples/             # Sample input files for testing
│   ├── docs/                    # Documentation
│   │   ├── api/                 # API documentation
│   │   ├── deployment/          # Deployment guides
│   │   ├── development/         # Development documentation
│   │   └── user-guides/         # User guides and tutorials
│   ├── knowledge/               # CrewAI knowledge base
│   │   └── user_preference.txt  # User context and preferences
│   ├── scripts/                 # Utility scripts
│   │   ├── extract_rules.py     # Rule extraction utilities
│   │   └── performance_benchmark.py # Performance testing
│   ├── src/                     # Source code
│   │   └── proof_reading_multiagent_system/  # Main package
│   │       ├── config/          # Agent and task configurations
│   │       ├── models/          # Data models and schemas
│   │       ├── tools/           # Custom CrewAI tools
│   │       ├── utils/           # Utilities and logging infrastructure
│   │       ├── config.yaml      # Main system configuration
│   │       ├── crew.py          # CrewAI crew definition
│   │       └── main.py          # Main execution entry point
│   ├── tests/                   # Test suite
│   │   ├── fixtures/            # Test fixtures and data
│   │   ├── integration/         # Integration tests
│   │   ├── performance/         # Performance tests
│   │   └── unit/               # Unit tests
│   ├── pyproject.toml          # Project configuration and dependencies
│   └── README.md               # Project-specific README
├── CLAUDE.md                   # Claude Code instructions (this file)
├── requirements.txt            # Python dependencies
└── README.md                   # Root-level README
```

### Data Processing Architecture
**CRITICAL**: The system is optimized to process only Medium/High severity issues for 60-80% efficiency gain.

#### Required Data Flow Pattern
```
XLSX Input → SeverityFilter → [Medium/High Issues] + [Low Issues (untouched)]
                ↓                                      ↓
        Process Medium/High Only              Keep Low Issues Separate
                ↓                                      ↓
        Updated Medium/High Issues ← DataMergeTool ← Untouched Low Issues
                ↓
        XLSX Output (Complete Dataset)
```

### Current Design
- **Medical Writer Team Lead (Reviewer Agent)**: Reviews document issues and flags potentially misclassified severity levels
- **Medical Writer (Reassessment Agent)**: Reassesses flagged issues using predefined rulesets and corrects severity assignments
- Excel integration for reading document issues and outputting corrected assessments
- Integration with Google VertexAI for language processing

## 3. Configuration Management

### Core Configuration Philosophy
**MANDATORY**: All configuration must be self-contained within config.yaml files. The system MUST NOT depend on users manually setting environment variables.

- **SELF-CONTAINED**: All configuration values must be specified in config.yaml
- **NO ENV DEPENDENCIES**: Users should never need to export environment variables manually
- **INTERNAL SETUP**: System should internally set environment variables from config values
- **USER-FRIENDLY**: Configuration should be accessible through single config.yaml modification

### Configuration Architecture Pattern
```python
# Required pattern for environment-independent configuration
def _setup_google_cloud_environment(self) -> None:
    """Set up Google Cloud environment from configuration instead of requiring environment variables."""
    try:
        gemini_config = self.config.get('gemini_config', {})
        credentials_path = gemini_config.get('credentials_path')
        if credentials_path:
            # Handle relative paths relative to config file location
            if not os.path.isabs(credentials_path):
                config_dir = os.path.dirname(__file__)
                credentials_path = os.path.join(config_dir, credentials_path)
            # Set environment variable internally
            os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = credentials_path
```

### Absolute Path Configuration Requirements
**MANDATORY**: All file and directory paths in config.yaml must use absolute paths for production stability.

- **LOG FILES**: All log file paths must be absolute (e.g., `/Users/gadappa/GenAI/genAI/proof_reading_multiagent_system/logs/`)
- **OUTPUT DIRECTORY**: Output directory must use absolute path to ensure consistent file generation
- **CREDENTIALS**: Google Cloud credentials path can use `$HOME` expansion for user directory
- **SESSION LOGS**: Session-specific log pattern must include full absolute path with variables

### Configuration Best Practices
- **ONE-FILE SETUP**: Users should only need to edit config.yaml, never set environment variables
- **ABSOLUTE PATHS**: Always use absolute paths for files and directories in production
- **PATH VALIDATION**: System validates all configured paths at startup
- **VALIDATION FEEDBACK**: Provide clear error messages when configuration is incomplete
- **ERROR HANDLING**: Graceful error messages for configuration issues

## 4. Development Workflow

### Document Context Configuration
Document processing context is configured in config.yaml for consistent processing:

```yaml
# Document processing context (no CLI override)
document_context:
  document_type: "Clinical Study Report"  # Type of document being processed
  domain: "Oncology"                     # Medical domain/specialty  
  urgency: "Standard"                    # Processing priority level

# Behavioral settings (optional CLI override)
behavior_config:
  verbose: false  # Enable detailed console output by default
```

### CLI Usage - Config-First Approach
The system uses a config-first approach where most settings are defined in config.yaml:

```bash
# Essential usage - all context from config
python -m proof_reading_multiagent_system.main --input "data/samples/issues.xlsx"

# Optional overrides
python -m proof_reading_multiagent_system.main --input "issues.xlsx" --output "/custom/path/result.xlsx"
python -m proof_reading_multiagent_system.main --input "issues.xlsx" --verbose
python -m proof_reading_multiagent_system.main --input "issues.xlsx" --quiet

# Development commands
run_crew
train
replay 
test
```

### Git Commit Guidelines
#### Atomic Commits at Feature Level
- **MANDATORY**: Each commit must represent one complete, working feature increment
- **ATOMIC PRINCIPLE**: Commits should be self-contained and represent logical units of work
- **FEATURE COMPLETENESS**: Each commit should leave the codebase in a functional state
- **INCREMENTAL PROGRESS**: Commit frequently at logical stopping points during development

#### Required Development Git Workflow Pattern
```bash
# 1. Activate virtual environment first
source .virtenv/bin/activate

# 2. Make code changes for a specific feature increment
# 3. Stage related files strategically
git add specific_files_for_feature

# 4. Commit with descriptive message
git commit -m "Implement [specific feature]: brief description"

# 5. Continue development cycle
```

#### Commit Message Standards
- **FORMAT**: `[Action] [Component/Area]: Brief description of what was accomplished`
- **EXAMPLES**: 
  - `Add ExcelReaderTool: implement XLSX to DocumentIssue conversion`
  - `Fix logging integration: resolve correlation ID threading issue`
  - `Update config: add Gemini 2.0-flash API settings`

#### Prohibited Practices
- **DO NOT** include Claude Code generation attributions in commit messages
- **DO NOT** add "Generated with Claude Code" or "Co-Authored-By: Claude" footers
- **DO NOT** make massive commits combining unrelated changes
- **DO NOT** commit broken or non-functional code

### Development Guidelines
- **VIRTUAL ENV**: Always activate .virtenv before any Python operations
- **GIT INTEGRATION**: Make staging and committing part of your regular development rhythm
- **PROGRESS TRACKING**: Use commits to demonstrate incremental progress and maintain project history
- **DOCUMENTATION FIRST**: Write comprehensive docstrings before or during implementation
- **TYPE SAFETY**: Use type hints extensively for better code maintainability and IDE support
- **MEMORY AWARENESS**: Monitor memory usage during development and follow memory management guidelines

## 5. Logging & Monitoring

### Mandatory Logging Integration
**CRITICAL**: All new components must use the established logging infrastructure.

- **ALWAYS** import and use logging utilities from `utils/` package
- **NEVER** use basic Python logging directly - use the structured logging system
- **ALWAYS** use correlation IDs for request tracking across components
- **REQUIRED** for all new components: Excel tools, rule engines, agent logic

### Required Logging Patterns
```python
# Correct pattern for new components
from proof_reading_multiagent_system.utils import (
    get_logger, log_structured, correlation_context,
    performance_monitor, audit_logger, log_execution_time
)

# Use context managers for performance tracking
with performance_monitor.track_processing_operation("operation_name") as metrics:
    # Processing logic here
    metrics.records_processed = count

# Use audit logging for any severity changes
audit_logger.log_severity_change(issue_id, old, new, reason, agent)

# Use performance decorators for critical functions
@log_execution_time("function_name")
def critical_function():
    pass
```

### Logging Categories
- `log_agent_decision()` - Agent flagging and reassessment decisions
- `log_excel_operation()` - File I/O operations with metadata
- `log_rule_application()` - Rule engine decision tracking
- `log_api_call()` - Gemini API usage and performance
- `log_performance()` - System performance metrics
- `log_error()` - Structured error logging with context
- `audit_logger.*` - Compliance and change tracking

### CrewAI Integration Standards
#### Session Management Requirements
- **ALWAYS** use `@before_kickoff` and `@after_kickoff` decorators for session logging
- **REQUIRED**: Generate unique session IDs for request tracking
- **PATTERN**: Follow the established `crew.py` logging integration pattern

```python
@CrewBase
class YourCrewSystem:
    def __init__(self):
        self.logger = setup_logging()
        self.session_id = generate_correlation_id()
        
    @before_kickoff
    def setup_session_logging(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        with correlation_context(self.session_id):
            self.logger.info("Starting session", extra={'session_id': self.session_id})
            return inputs
            
    @after_kickoff  
    def finalize_session_logging(self, output: Any) -> Any:
        performance_monitor.log_session_summary()
        return output
```

## 6. Quality & Testing Standards

### Error Handling Standards
- **NEVER** use generic try/catch without structured logging
- **REQUIRED**: Use `log_error()` with error context and correlation IDs
- **PATTERN**: Provide graceful degradation with informative logging

```python
# Required error handling pattern
try:
    # Operation here
    pass
except Exception as e:
    log_error(
        f"Operation failed: {str(e)}",
        error_type=type(e).__name__,
        operation="operation_name",
        correlation_id=get_correlation_id(),
        additional_context={"key": "value"}
    )
    raise  # Re-raise after logging
```

### Performance Monitoring Requirements
- **REQUIRED**: Track processing times for all critical operations
- **REQUIRED**: Use `@log_execution_time` decorators for performance-critical functions

### Testing Standards
- **REQUIRED**: Create comprehensive test suites for any new infrastructure
- **VALIDATION**: Always include environment setup and fallback testing
- **COVERAGE**: Test both success and failure scenarios with proper logging

### Development Notes
- The project structure follows CrewAI project conventions with nested directory structure
- Agents are defined as methods with `@agent` decorator returning `Agent` objects
- Tasks are defined as methods with `@task` decorator returning `Task` objects  
- The crew uses `Process.sequential` execution by default
- Custom tools should extend `BaseTool` and implement `_run` method with proper Pydantic schema

## 7. Documentation Standards

### Documentation Requirements (PEP 257 Compliance)
- **MANDATORY**: All modules, classes, functions, and methods must have comprehensive docstrings
- **PEP 257**: Follow Python Enhancement Proposal 257 for docstring conventions
- **GOOGLE STYLE**: Use Google-style docstrings for consistency with project tooling
- **SPHINX COMPATIBLE**: Ensure all documentation is Sphinx-compatible for auto-generation

### Required Documentation Patterns

#### Module-Level Documentation
```python
"""
Comprehensive module description following PEP 257.

This module provides [brief description]. Key components include:
    - ClassName: Brief description of main classes
    - function_name(): Brief description of key functions
    
Example:
    Basic usage example::
    
        from proof_reading_multiagent_system.module_name import ClassName
        instance = ClassName()
        result = instance.method()

Note:
    Any important notes about module usage, dependencies, or limitations.
"""
```

#### Function Documentation Standards
```python
def process_excel_file(
    file_path: str, 
    severity_filter: Optional[List[str]] = None
) -> Dict[str, Any]:
    """
    Process Excel file and extract DocumentIssue objects with filtering.
    
    Args:
        file_path: Absolute path to the Excel file to process.
        severity_filter: Optional list of severity levels to include.
            
    Returns:
        Dictionary containing processed results with structure::
        
            {
                "issues": List[DocumentIssue],
                "metadata": {"total_issues": int, "efficiency_gain": float}
            }
            
    Raises:
        FileNotFoundError: If the specified Excel file does not exist.
        ValueError: If the file format is invalid or missing required columns.
        
    Example:
        Basic usage with severity filtering::
        
            result = process_excel_file(
                "/path/to/issues.xlsx",
                severity_filter=["High", "Medium"]
            )
    """
```

### Type Annotation Standards
```python
from typing import Dict, List, Optional, Union, Any, Tuple
from pathlib import Path
from datetime import datetime

# Type aliases for complex types
IssueDict = Dict[str, Union[str, int, datetime]]
ProcessingResult = Tuple[List[DocumentIssue], Dict[str, Any]]
SeverityLevel = Literal["High", "Medium", "Low"]
```
### Documentation Maintenance
- **UPDATE REQUIREMENT**: Update documentation in the same commit as code changes
- **REVIEW PROCESS**: Documentation changes must be reviewed for accuracy and completeness
- **CONSISTENCY**: Maintain consistent style and format across all documentation

## 8. Production Considerations

### Key Dependencies
- `crewai[tools]>=0.165.1,<1.0.0` - Core multiagent framework
- `google-cloud-aiplatform>=1.36.0` - VertexAI integration for Gemini 2.0-flash
- `google-cloud-logging>=3.8.0` - Google Cloud Logging for monitoring and audit trails
- `pandas>=2.0.0` - Excel data processing
- `openpyxl>=3.1.0` - Excel file read/write operations
- `pydantic>=2.0.0` - Data model validation
- `pyyaml>=6.0` - Configuration file processing
- Python 3.10+ requirement

### CrewAI Documentation Reference
- **Latest CrewAI Documentation**: `/Users/gadappa/GenAI/genAI/crewai-docs/docs/en` - Local copy of official CrewAI documentation for reference

### Google Cloud Integration Requirements
#### Environment Configuration
- **FALLBACK**: System gracefully degrades to console logging if cloud services unavailable

#### API Cost Monitoring
- **MANDATORY**: Use `performance_monitor.track_api_call()` for all Gemini API calls
- **REQUIRED**: Log token usage and cost estimates for optimization
- **PATTERN**: Track API performance metrics for cost control

```python
# Required pattern for all Gemini API calls
with performance_monitor.track_api_call("gemini") as metrics:
    # Make API call here
    result = llm.generate(prompt)
    metrics.tokens_used = result.token_count
    metrics.cost_estimate = calculate_cost(result.token_count)
```

### Production Deployment Requirements
- **REQUIRED**: Separate configuration for development, staging, and production
- **VALIDATION**: Environment-specific validation before deployment
- **TESTING**: Comprehensive smoke tests in each environment
- **MONITORING**: Resource utilization tracking and auto-scaling configuration

### Knowledge System
- `knowledge/user_preference.txt` contains user context (AI Engineer, San Francisco, interested in AI Agents)
- CrewAI knowledge integration is supported but not yet implemented

## Development Guidelines Summary
- When done with planning a project, create a detailed markdown file with implementation plan
- Always use scratch pads and checklists while implementing the project as per the plan
- **VIRTUAL ENV**: Always activate .virtenv before any Python operations
- **GIT INTEGRATION**: Make staging and committing part of your regular development rhythm
- **PROGRESS TRACKING**: Use commits to demonstrate incremental progress and maintain project history
- **DOCUMENTATION FIRST**: Write comprehensive docstrings before or during implementation
- **TYPE SAFETY**: Use type hints extensively for better code maintainability and IDE support
- **MEMORY AWARENESS**: Monitor memory usage during development and follow memory management guidelines

---

**Note**: For detailed deployment procedures, security frameworks, and extensive code examples, refer to:
- `CLAUDE_DETAILED_BACKUP.md` - Full original documentation
- `DEPLOYMENT_GUIDE.md` - Production deployment procedures  
- `DOCUMENTATION_PATTERNS.md` - Detailed documentation examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gprakhar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
