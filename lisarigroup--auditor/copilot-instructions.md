## auditor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AWS Infrastructure Security Auditor v2.1 - A comprehensive Python tool for auditing AWS infrastructure security vulnerabilities, optimizing costs, and generating automated remediation scripts. The tool performs read-only analysis by default and generates reviewable bash scripts for any modifications.

## Essential Development Commands

### Setup and Installation
```bash
make setup                    # Complete environment setup (recommended for first use)
make install                  # Install Python dependencies only
make check-aws               # Verify AWS credentials configuration
```

### Production Workflow (Read-only and Safe)
```bash
make prod-fetch              # Download AWS resource data
make prod-audit              # Perform security analysis on fetched data
make prod-dashboard          # Launch Streamlit dashboard
make prod-full               # Complete fetch + audit + dashboard workflow
```

### Specialized Analysis Commands
```bash
make sgc                     # Security Groups + Cost analysis (shortcut)
python main.py --sg-cost-analysis     # Full Security Groups with cost integration
python main.py --vpc-analysis         # Comprehensive VPC topology analysis
python main.py --network-optimization # Network cost optimization analysis
```

### Testing and Development
```bash
make prod-quick              # Quick test on existing data
make prod-single             # Single region test (us-east-1 only)
python -m pytest            # Run test suite (when tests are added)
```

### Code Quality
```bash
black .                      # Format code
flake8 .                     # Lint code
mypy .                       # Type checking
```

## Architecture Overview

### Core Flow
1. **Fetch Phase** (`utils/async_fetcher.py`): Asynchronously collects AWS resource data using boto3/aioboto3
2. **Audit Phase** (`audit/audit_engine.py`): Orchestrates specialized auditors to analyze security and cost issues
3. **Report Phase**: Generates findings in multiple formats and creates remediation scripts

### Key Components

- **Main Entry Point** (`main.py`): CLI interface with multiple operation modes
- **Configuration System** (`config/settings.py`, `config/audit_rules.py`): AWS settings and security rules
- **Audit Engine** (`audit/audit_engine.py`): Orchestrates all auditors, returns Finding objects
- **Specialized Auditors** (`audit/`): EC2, Security Groups, VPC, S3, etc. - each inherits from `BaseAuditor`
- **Utilities** (`utils/`): Cost analysis, data processing, Security Group optimization
- **Dashboard** (`dashboard/`): Streamlit web interface for interactive analysis

### Data Structure
- **Raw AWS Data**: Stored in `data/` as JSON files (gitignored)
- **Reports**: Generated in `reports/` with markdown summaries and bash scripts
- **Findings**: Standardized `Finding` dataclass with severity, description, and remediation info

### Key Architectural Patterns

1. **Modular Auditors**: Each auditor inherits from `BaseAuditor` and implements `audit()` method returning `Finding` objects
2. **Async Data Fetching**: Uses `aioboto3` for parallel AWS API calls across regions/services
3. **Cost Integration**: Specialized tools integrate AWS Cost Explorer data with security findings
4. **Read-Only by Default**: All operations are safe; modifications only via generated scripts

## Configuration

### AWS Configuration (`config/settings.py`)
- Supports multiple AWS regions via `--regions` flag
- Service selection via `--services` flag  
- Uses environment variables for AWS credentials
- Centralized timeout and retry settings

### Security Rules (`config/audit_rules.py`)
- Dangerous ports definitions (SSH: 22, RDP: 3389, etc.)
- Compliance framework mappings
- Risk severity classifications

## Specialized Features

### Security Groups + Cost Analysis
- Unique integration of security analysis with real AWS cost data
- ROI calculations for security group optimizations
- Executive-level reporting with financial impact

### VPC Analysis (`--vpc-analysis`)
- Complete network topology mapping
- Subnet analysis and optimization recommendations
- Network ACL and routing analysis

### Automated Remediation
- Generates bash scripts in `reports/cleanup/`
- Includes backup procedures before modifications
- Verification steps included in all scripts

## Testing Strategy

Uses pytest framework. When adding tests:
- Place test files in `tests/` directory
- Use `test_` prefix for test files and functions
- Mock AWS API calls using moto library for unit tests
- Run with `python -m pytest`

## Important Development Notes

- **Never modify AWS resources directly**: All changes via generated scripts only
- **Cost data requires permissions**: AWS Cost Explorer access needed for cost analysis features
- **Multi-region support**: Most auditors work across regions specified in `--regions`
- **Finding object consistency**: All auditors return standardized `Finding` objects with severity, resource_id, description, and remediation fields

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lisarigroup)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lisarigroup)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
