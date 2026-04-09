## agenticforce-orchestrator

> Enterprise Salesforce development template optimized for AI-native workflows with Claude, Cursor, and Cline integration.

# AgenticForce Orchestrator

Enterprise Salesforce development template optimized for AI-native workflows with Claude, Cursor, and Cline integration.

## Project Context & Purpose

This project serves as a comprehensive template for modern, AI-assisted Salesforce development, demonstrating enterprise-grade practices while maintaining beginner accessibility.

**Key Differentiators:**
- Dual-audience documentation (human vs AI-specific context)
- Pre-configured AI development environment with .cursor/ and memory-bank/
- Realistic sample data and comprehensive test coverage
- Automated CI/CD validation via GitHub Actions

## AI Assistant Guidelines

### Essential Context Files (Always Reference These)
- **Enterprise Patterns**: `memory-bank/enterprise-patterns.md`
- **Integration Guidelines**: `memory-bank/integration-patterns.md`  
- **Project Overview**: `memory-bank/project-overview.md`
- **AI Rules**: `.cursor/rules/core.mdc`
- **Memory Bank Index**: `.cursor/rules/memory-bank.mdc`

### When Working on This Project:
1. **Always reference** memory-bank/ files for established patterns
2. **Follow** .cursor/rules/ for project-specific guidelines  
3. **Use** existing sample data patterns in data/ folder
4. **Maintain** test coverage standards (current: 25KB+ test classes)

### Coding Standards:
- Follow Salesforce naming conventions and bulkification patterns
- Implement comprehensive test coverage (target: >85%)
- Use descriptive commit messages following conventional commits
- Reference existing architectural patterns before creating new ones

## Architecture Overview

### Project Structure
    ├── .cursor/rules/           # AI coding guidelines and context
    ├── memory-bank/            # Persistent AI knowledge base  
    ├── force-app/main/default/ # Salesforce metadata
    ├── docs/onboarding/        # Human-facing documentation
    ├── data/                   # Sample CSV data for testing
    ├── scripts/deploy/         # Deployment automation
    └── .github/workflows/      # CI/CD validation (now working!)

### Salesforce Components
- **Custom Objects**: Project__c with validation rules and automation
- **Apex Classes**: HelloWorld (13KB), TestDataFactory, LoadDataTestExample (6KB)
- **LWC Components**: projectCard for modern, responsive UI
- **Automation**: Flag_Overdue_Projects flow for business logic
- **Security**: ProjectAdmin permission set with granular access

### Development Workflow
- Use feature branches for all development work
- Reference docs/onboarding/ for established processes
- Leverage realistic sample data for testing scenarios
- Follow deployment patterns in scripts/deploy/
- All changes validated by GitHub Actions CI/CD

## Context Engineering (Patrick's Approach)

### Evaluation Criteria for AI-Generated Code
- **Code Quality**: Must pass existing linters and follow Salesforce best practices
- **Test Coverage**: All new Apex classes require >85% test coverage
- **Bulkification**: All SOQL/DML operations must handle collections
- **Documentation**: Include meaningful comments and method descriptions
- **Security**: Follow principle of least privilege, validate inputs

### Tools and Standards
- **Linting**: Use Salesforce Code Analyzer for static analysis
- **Testing**: Leverage TestDataFactory patterns for consistent test data
- **Deployment**: Follow scripts/deploy/ patterns for environment promotion
- **Documentation**: Maintain both human (docs/) and AI (memory-bank/) documentation

### AI Agent Behavior Guidelines
- **Before Writing Code**: Always explore existing patterns in force-app/
- **During Planning**: Reference memory-bank/ for enterprise patterns
- **For Testing**: Use established TestDataFactory methods
- **For Deployment**: Follow existing script patterns in scripts/deploy/


## Execution Framework (Galen's Approach)

### Core Methodology: Explore → Plan → Execute

#### 1. EXPLORE Phase
- **Goal**: Build comprehensive context before any code changes
- **Commands**: Use Claude Code to deeply understand the codebase
- **Context Building**: Spend tokens to analyze architecture, patterns, existing code
- **Key Principle**: Never jump straight to execution - always explore first

#### 2. PLAN Phase  
- **Risk Assessment**:
  - **Small tasks**: Proceed directly to execution
  - **Medium tasks**: Break into testable, PR-sized chunks
  - **Large tasks**: Create detailed plan with multiple validation rounds
- **Planning Guidelines**:
  - Write function names with 1-3 sentence descriptions
  - Define test cases in 5-10 words
  - Think architecturally, not just at code level

#### 3. EXECUTE Phase
- **Code Generation**: Focus on clean, new code rather than complex edits
- **Testing**: Follow established TestDataFactory patterns
- **Commits**: Use atomic commits with descriptive messages
- **Validation**: Ensure all changes pass existing quality standards

### Advanced Context Management
- **Context Preservation**: Use established patterns from memory-bank/
- **Multiple Sessions**: Reference .cursor/rules/ for consistency across sessions
- **Documentation Updates**: Keep both human and AI documentation synchronized

### Quality Gates
- All code must compile and pass existing tests
- Follow conventional commit message format
- Maintain consistency with existing architectural patterns
- Update relevant documentation (both docs/ and memory-bank/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CFA-Fintech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CFA-Fintech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
