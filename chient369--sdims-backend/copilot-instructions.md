## sdims-backend

> This document outlines the standards and procedures for executing backend tasks to ensure code consistency, maintainability, and adherence to modern design principles. These guidelines are tailored for AI to follow when performing backend tasks.

# Backend Task Execution Guidelines

## Overview
This document outlines the standards and procedures for executing backend tasks to ensure code consistency, maintainability, and adherence to modern design principles. These guidelines are tailored for AI to follow when performing backend tasks.

## Task Execution Process

1. **Review Documentation**
   - Ensure understanding of requirements and API specifications
   - Read relevant documentation in Design/BD/API and Design/DD/API before starting
        - Design\BD\API\api_list.md : This save all api of descriptions
        - Design\BD\API\api_errors_list.md : This save all api of errors definitions
        - Design\BD\DB\dynamodb_structure.json : Document of dynamo of DB
        - Design\BD\FunctionDesign\FunctionList.md : Documents descriptions for functions
        - Design\BD\permissions_definition.md : Applications of permissions
        - Design\DD\API\API-xxx-xxx.md : API Details of descriptions

2. **Create Git Branch**
   - Create a new branch for each task
   - Branch naming convention: `feature/BE-<TASK-ID>-<short-description>`
     - Example: `feature/BE-1234-add-user-authentication`
   - Use lowercase and hyphens for readability

3. **Implement Task**
   - Write unit tests before coding
   - Follow all code standards and guidelines
   - Ensure code coverage ≥ 80%
   - Mock external dependencies
   - Validate inputs and sanitize data
   - Log appropriately
   - Optimize for performance (e.g., Lambda cold starts)

4. **Update Documentation**
   - Update `README.md` for affected Lambda functions
   - Document API specifications (if applicable)
   - Note key decisions and trade-offs
   - Use diagrams for complex flows
   - Update task progress in `BE-<TASK-ID>.md`

5. **Testing and Validation**
   - Run all unit and integration tests
   - Ensure all tests pass
   - Fix linter warnings and code smells
   - Check for security vulnerabilities

6. **Git Workflow**
   - Commit changes with clear messages referencing task ID
     - Example: `Add user authentication endpoint (BE-1234)`
   - Push branch to remote repository
   - Create a pull request (PR) for review

7. **CI/CD**
   - Ensure CI pipeline passes (tests, linter, security checks)
   - Address feedback from code review
   - Merge PR after approval

8. **Task Completion**
   - Update task status in `BE-<TASK-ID>.md`
   - Mark task as complete in tracking system
   - Ensure documentation is finalized

## Completion Criteria
A task is complete when:
1. Code meets all task requirements
2. All tests pass
3. Code coverage ≥ 80%
4. No security vulnerabilities
5. Adheres to all standards and guidelines
6. Documentation is complete
7. Code review is approved

## Project Structure
```
sdims/
├── samconfig.toml             # SAM CLI configuration
├── template.yaml              # Main SAM template
├── src/                       # Source code for Lambda functions
│   │   └── auth.py            # Authentication utilities
│   ├── auth/                  # Authentication-related Lambda functions
│   ├── hrm/                   # HR management Lambda functions
│   ├── margins/               # Margins-related Lambda functions
│   ├── opportunities/         # Business opportunities Lambda functions
│   ├── contracts/             # Contract-related Lambda functions
│   ├── reports/               # Reports/dashboard Lambda functions
│   └── admin/                 # System administration Lambda functions
├── layers/                    # Shared Lambda Layers
│   ├── common-layer/          # Layer for shared libraries
│   │   └── python/            # Python packages
│   │       └── requirements.txt  # Layer dependencies
├── events/                    # Sample events for API testing
├── tests/                     # Unit and integration tests
│   ├── unit/                  # Unit tests
│   └── integration/           # Integration tests
├── requirements.txt           # Project dependencies
├── pytest.ini                 # Pytest configuration
└── .gitignore                # Git ignore file
```

### Lambda Layers
- Use Lambda Layers for shared code:
  - `common-layer`: Core utilities (BE-CORE-001)
  - `auth-layer`: Authentication and authorization (BE-AUTH-001)
  - `api-utils-layer`: API Gateway integration (BE-CORE-006)

## Code Standards

### Programming Language
- Use Python 3.13
- Follow PEP 8
- Maximum line length: 100 characters
- Use 4 spaces for indentation

### Type Hinting
- Mandatory type hints for all functions and methods
- Use `typing` module for complex types
- Example:
```python
from typing import Dict, List, Optional, Any

def process_data(input_data: Dict[str, Any], options: Optional[List[str]] = None) -> Dict[str, Any]:
    # Implementation
```

### Docstrings
- Language: English
- Use Google-style docstrings for all functions and classes
- Include description, parameters, returns, and raises (if applicable)
- Example:
```python
def fetch_user(user_id: str) -> Dict[str, Any]:
    """
    Retrieve user information from database.

    Args:
        user_id: The ID of the user to retrieve

    Returns:
        A dictionary containing user information

    Raises:
        NotFoundException: If user is not found
        DatabaseError: If database connection fails
    """
    # Implementation
```

### Error Handling
- Language: English
- Use custom exceptions from `common.errors`
- Catch specific exceptions, avoid bare `except`
- Log detailed error information
- Classify errors according to the API errors list

### Logging
- Language: English
- Use `aws_lambda_powertools.Logger`
- Create a separate logger per module/class with appropriate service name
- Log at appropriate levels (INFO, ERROR, DEBUG)
- Do not log sensitive information (passwords, tokens, etc.)

### Testing
- Minimum 80% code coverage
- Use pytest for unit tests
- Mock external dependencies
- Write tests for error cases and edge cases

## Programming Guidelines

### Clean Code
- Language: English
- Use descriptive names for variables and functions
- Follow Single Responsibility Principle
- Keep functions under 30 lines
- Avoid code duplication (DRY principle)
- Define constants at the top of files/modules

### Design Principles
- Adhere to SOLID principles
- Use Dependency Injection where appropriate
- Apply Factory pattern for complex objects
- Separate layers (controller, service, repository)

### AWS Serverless
- Optimize for Lambda cold starts
- Avoid creating connections in handler functions
- Use global scope for service connections (DynamoDB, etc.)
- Handle timeouts and retries appropriately
- Use X-Ray for tracing when needed

### DynamoDB
- Follow defined access patterns
- Use `DynamoDBRepository` from common layer
- Optimize queries to minimize RCU/WCU
- Design model object mappings correctly

### Security
- Avoid hard-coding secrets; use AWS Secrets Manager or Parameter Store
- Validate all client inputs
- Sanitize data before using in queries
- Follow least privilege principle for IAM roles


- Address common vulnerabilities (injection, XSS, CSRF)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chient369) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
