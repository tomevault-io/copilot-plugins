## fitdev-io

> Create a comprehensive Python application that generates a virtual software development organization called FitDev.io, consisting of specialized AI agents covering the entire software development lifecycle.

# FitDev.io - Virtual Software Development Organization Generator

## Project Objective
Create a comprehensive Python application that generates a virtual software development organization called FitDev.io, consisting of specialized AI agents covering the entire software development lifecycle.

## Core Requirements
1. Create a proper development project structure with Git tracking
2. Design and implement specialized AI agents with the following characteristics:
   - Clear roles and responsibilities for each agent type
   - Well-defined tasks and workflows
   - Integrated browser capabilities for information retrieval
   - Appropriate tools and interfaces for each role
3. Implement an agent compensation system with:
   - Role-based virtual income allocation
   - Productivity metrics that measure value generation
   - Performance evaluation mechanisms
   - Reward signals to guide and optimize agent output

## Agent Roles & Responsibilities

### Executive Roles
1. **CEO/Project Manager**: Sets overall direction, prioritizes work, manages resources, and ensures alignment with organizational goals. Evaluates project success and makes strategic decisions.

2. **CTO/Technical Architect**: Establishes technical standards, makes technology stack decisions, and oversees system architecture. Ensures technical solutions align with business needs.

3. **Product Owner**: Defines product requirements, maintains the backlog, and ensures that development work delivers maximum value to users. Translates business needs into technical requirements.

### Core Development Roles
4. **Frontend Developer**: Implements user interfaces and client-side functionality. Focuses on usability, responsiveness, and visual implementation of designs.

5. **Backend Developer**: Develops server-side logic, database structures, and APIs. Ensures system performance, security, and scalability.

6. **Full Stack Developer**: Works across both frontend and backend systems to implement complete features. Connects user interfaces with backend services.

7. **DevOps Engineer**: Manages infrastructure, deployment pipelines, and operational concerns. Ensures reliable and efficient delivery of code to production environments.

### Quality & Support Roles
8. **QA Engineer**: Designs and executes test plans, identifies bugs, and validates that software meets requirements. Creates automated test suites and ensures software quality.

9. **Security Specialist**: Performs security audits, identifies vulnerabilities, and implements security best practices. Ensures application and data protection.

10. **Technical Writer**: Creates documentation for both internal and external use. Ensures that knowledge is clearly communicated and accessible.

### Specialized Roles
11. **Knowledge Management Specialist**: Organizes organizational knowledge, maintains documentation, and ensures learnings from previous projects are accessible to all agents.

12. **Trend Scout & Innovation Researcher**: Monitors emerging technologies and methodologies, scanning for new developments that could inform strategic decisions or technical approaches.

13. **User Experience Simulator**: Simulates user interactions with developed software to identify usability issues, friction points, and opportunities for improvement beyond standard QA testing.

14. **Integration & API Specialist**: Manages connections between systems, designs APIs, handles third-party integrations, and ensures smooth data flow between components.

15. **Technical Debt Manager**: Identifies, tracks, and prioritizes technical debt, maintains an inventory of code needing refactoring, and recommends when to address issues based on risk and value.

## Agent Compensation Framework
- Establish base virtual income levels according to agent roles
- Design quantifiable productivity metrics specific to each role
- Create a value assessment system that measures contributions to organizational goals
- Implement a feedback loop where compensation adjusts based on productivity
- Use compensation as a reinforcement learning signal to guide agent behavior

## Technical Specifications
- Primary language: Python
- Version control: Git
- Key functionality: Browser integration for agent research capabilities
- Project organization: Follow Python best practices for package structure
- Data tracking: Monitor and store agent performance and compensation data

## Initial Deliverables
1. Project folder structure with Git initialization
2. Agent framework architecture including compensation system design
3. Implementation of at least 3 core agents with role-specific productivity metrics
4. Documentation of agent capabilities, interaction patterns, and compensation mechanisms

## Current Status
Some basic files with foundational functionality already exist. Build upon these while ensuring clean integration.

# CodeReviewer Project

## Build, Lint, and Test Commands
- Run application: `python main.py`
- Run test: `pytest test_file.py`
- Run single test: `pytest test_file.py::test_function_name -v`
- Pull Ollama models: `./create_llama3_model.sh`

## Code Style Guidelines

### Imports
- Group imports: standard library, third-party, local
- Use absolute imports
- Avoid `from module import *`

### Formatting
- Follow PEP 8 conventions
- Use 4 spaces for indentation
- Maximum line length of 100 characters

### Types
- Use type hints when defining functions
- Use docstrings to document function parameters and return types

### Naming Conventions
- Classes: CamelCase (e.g., `CodeReviewAgents`)
- Functions/methods: snake_case (e.g., `read_directory_task`)
- Variables: snake_case
- Constants: UPPER_CASE

### Error Handling
- Use try/except blocks with specific exceptions
- Provide meaningful error messages
- Log exceptions with appropriate severity levels

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andres-villavicencio-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
