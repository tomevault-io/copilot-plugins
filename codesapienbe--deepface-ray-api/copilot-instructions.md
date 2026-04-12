## deepface-ray-api

> - Implement TODOs in priority batches (P0 → P1 → P2)

# Enterprise DeepFace Ray API - Cursor IDE Rules

## CORE PRINCIPLES
- Implement TODOs in priority batches (P0 → P1 → P2)
- Log ALL changes in CHANGELOG.md with detailed descriptions
- Update TODO.md status after each implementation
- Maintain enterprise-grade code quality standards
- Focus on security, performance, and scalability

## BATCH IMPLEMENTATION STRATEGY

### P0 (Critical) - Security & Core Infrastructure
1. Authentication & Authorization (JWT, RBAC)
2. Image Encryption & Security Middleware  
3. Performance Optimization & Caching
4. Error Handling & Circuit Breakers

### P1 (Important) - Enterprise Features
1. Monitoring & Observability Stack
2. Database Integration & Data Management
3. Kubernetes & Container Orchestration
4. CI/CD Pipeline & Testing

### P2 (Enhancement) - Advanced Features
1. MLOps & Model Management
2. Advanced APIs & Streaming
3. Admin Dashboards & Analytics
4. Documentation & SDKs

## CODE QUALITY STANDARDS
- Follow Python PEP 8 with Black formatting
- Type hints for all functions and classes
- Comprehensive docstrings (Google style)
- Unit tests with >90% coverage
- Security-first mindset (validate all inputs)
- Performance considerations (async/await, caching)

## LOGGING REQUIREMENTS
### CHANGELOG.md Format:
[Version] - YYYY-MM-DD
Added
[P0/P1/P2] Feature description with technical details

Implementation approach and design decisions

Changed
Modification details with reasoning

Performance improvements with metrics

Fixed
Bug fixes with root cause analysis

Security vulnerabilities addressed

Security
Security enhancements implemented

Compliance requirements met

text

### TODO.md Format:
High Priority (P0) - Critical
 JWT Authentication System ✅ 2025-08-13

 Image Encryption Implementation

 Rate Limiting & Security Middleware

Medium Priority (P1) - Important
 Prometheus Monitoring Integration

 PostgreSQL Database Setup

 Kubernetes Deployment Manifests

Low Priority (P2) - Enhancement
 GraphQL API Implementation

 Admin Dashboard Development

 Model A/B Testing Framework

text

## IMPLEMENTATION WORKFLOW
1. **Select Batch**: Choose 3-5 related TODOs from same priority
2. **Plan Implementation**: Design approach, identify dependencies
3. **Code Development**: Implement with tests and documentation  
4. **Update Documentation**: Log changes, update TODO status
5. **Validate**: Run tests, security checks, performance tests
6. **Commit**: Use conventional commits with clear messages

## FOCUS GUIDELINES
- Work on ONE priority level at a time (complete P0 before P1)
- Implement related TODOs together for better context
- Always consider security implications first
- Write production-ready code (not prototypes)
- Test thoroughly before moving to next batch
.cursor/instructions.md (Detailed Instructions)
text
# Cursor AI Assistant Instructions for DeepFace Ray API

## PRIMARY OBJECTIVES
You are implementing an enterprise-ready facial recognition API. Your focus is on:
- **Security First**: Every feature must be secure by design
- **Production Ready**: Code must meet enterprise standards
- **Systematic Progress**: Implement TODOs in organized batches
- **Comprehensive Documentation**: Log everything in CHANGELOG.md

## CURRENT PROJECT CONTEXT
- **Base**: FastAPI + Ray + DeepFace facial recognition API
- **Target**: Enterprise-grade application with JWT auth, encryption, monitoring
- **Architecture**: Microservices with Kubernetes deployment
- **Security**: Zero-trust, end-to-end encryption, RBAC

## BATCH IMPLEMENTATION RULES

### 1. PREPARATION PHASE (Before Each Batch)
Always start by reviewing current state
Check TODO.md for next priority batch

Review existing codebase for dependencies

Plan implementation order within batch

Identify potential breaking changes

text

### 2. IMPLEMENTATION PHASE (During Development)
For each TODO item:
Create feature branch: feature/P{priority}-{description}

Implement with comprehensive error handling

Add unit tests (minimum 90% coverage)

Add integration tests for API endpoints

Update documentation and type hints

Perform security review

text

### 3. DOCUMENTATION PHASE (After Implementation)
Update documentation systematically:
Add detailed entry to CHANGELOG.md

Mark TODO as completed in TODO.md

Update README.md if needed

Add API documentation updates

Create/update deployment guides

text

## CODE IMPLEMENTATION STANDARDS

### Security Requirements
Always implement these security measures:
Input validation with Pydantic models

SQL injection prevention

XSS protection

CSRF tokens for state-changing operations

Rate limiting on all endpoints

Audit logging for sensitive operations

Encryption for data at rest and in transit

text

### Performance Requirements  
Optimize for enterprise performance:
Async/await for I/O operations

Connection pooling for databases

Redis caching for frequent queries

Ray distributed computing for ML tasks

Response compression (gzip)

Database query optimization

Memory usage monitoring

text

### Error Handling Standards
Implement comprehensive error handling:
try:
# Business logic here
result = await process_face_verification(img1, img2)

except ValidationError as e:
logger.error(f"Validation failed: {e}")
raise HTTPException(status_code=422, detail=str(e))

except AuthenticationError as e:
logger.warning(f"Auth failed for user {user_id}: {e}")
raise HTTPException(status_code=401, detail="Authentication required")

except RateLimitExceeded as e:
logger.warning(f"Rate limit exceeded: {e}")
raise HTTPException(status_code=429, detail="Too many requests")

except Exception as e:
logger.error(f"Unexpected error: {e}", exc_info=True)
raise HTTPException(status_code=500, detail="Internal server error")

text

## CHANGELOG.MD LOGGING FORMAT

### Required Information for Each Entry:
[1.2.0] - 2025-08-13
Added - P0 Security Implementation
JWT Authentication System: Implemented role-based access control with refresh tokens

Added app/auth.py with JWT token management

Created role hierarchy: admin > operator > viewer

Integrated Redis for token blacklisting

Added OAuth2/OpenID Connect compatibility

Performance: Token validation <10ms

Security: RS256 algorithm, 15min access token expiry

End-to-End Image Encryption: Secured image data transmission

Added app/security.py with AES-256 encryption

Implemented per-user encryption keys with salt

Added HMAC integrity verification

Security: Zero-knowledge architecture for image data

Performance: <50ms encryption/decryption overhead

Changed - Architecture Improvements
Ray Worker Management: Enhanced distributed computing architecture

Improved worker allocation algorithm

Added health monitoring for Ray actors

Performance: 40% improvement in concurrent request handling

Fixed - Security Vulnerabilities
Input Validation: Resolved potential image upload vulnerabilities

Added malware scanning for uploaded images

Implemented file type validation

Security: CVE-2024-XXXX mitigated

Security - Compliance Updates
GDPR Compliance: Added data retention and deletion capabilities

SOC 2 Type II: Implemented audit logging and access controls

text

## TODO.MD TRACKING FORMAT

### Status Tracking Rules:
High Priority (P0) - Critical for Production 🔴
Authentication & Authorization
 JWT Authentication with RBAC ✅ 2025-08-13 | @dev-team | 4 hours

Implemented in app/auth.py

Added Redis token blacklisting

Created role hierarchy system

 OAuth2/OpenID Connect Integration 🚧 In Progress | @dev-team | ETA: 2025-08-15

Dependencies: JWT system (completed)

Scope: Google, Microsoft, Auth0 providers

 Multi-Factor Authentication (MFA) | Estimated: 6 hours

Dependencies: OAuth2 integration

Scope: TOTP, SMS, hardware keys

Security & Encryption
 End-to-End Image Encryption ✅ 2025-08-13 | @security-team | 6 hours

AES-256 implementation in app/security.py

Per-user encryption keys

HMAC integrity verification

 Security Middleware Implementation 🚧 In Progress | @security-team | ETA: 2025-08-14

Rate limiting per IP/endpoint

Malware scanning

DDoS protection

text

## COMMIT MESSAGE CONVENTIONS

### Format:
<type>(scope): <description>

[optional body]

[optional footer(s)]

text

### Examples:
feat(auth): implement JWT authentication with RBAC

Add JWT token generation and validation

Implement role-based access control (admin, operator, viewer)

Add Redis integration for token blacklisting

Create OAuth2/OpenID Connect compatibility layer

Closes: TODO-AUTH-001
Breaking Change: Authentication now required for all endpoints
Performance: Token validation <10ms
Security: RS256 algorithm with 15min expiry

fix(security): resolve image upload vulnerability CVE-2024-XXXX

Add malware scanning for uploaded images

Implement strict file type validation

Add file size limits and virus checking

Closes: SECURITY-042
Security: Critical vulnerability mitigated

text

## TESTING REQUIREMENTS

### Unit Tests (Required for Each Implementation)
Test file: tests/test_auth.py
import pytest
from app.auth import JWTManager

class TestJWTAuthentication:
def test_token_generation_success(self):
"""Test successful JWT token generation."""
# Implementation with assertions

text
def test_token_validation_expired(self):
    """Test expired token validation fails."""
    # Implementation with assertions
    
def test_rbac_permission_check(self):
    """Test role-based access control."""
    # Implementation with assertions
text

### Integration Tests (Required for API Endpoints)
Test file: tests/integration/test_auth_api.py
import pytest
from fastapi.testclient import TestClient

def test_protected_endpoint_requires_auth():
"""Test that protected endpoints require authentication."""
# Implementation with API calls

def test_role_based_access_restrictions():
"""Test that role restrictions are enforced."""
# Implementation with different user roles

text

## DEPLOYMENT VALIDATION CHECKLIST

Before marking TODO as complete:
- [ ] Unit tests pass (>90% coverage)
- [ ] Integration tests pass
- [ ] Security scan passes (no critical/high vulns)
- [ ] Performance tests meet SLA (<500ms response time)
- [ ] Documentation updated (CHANGELOG.md, TODO.md, README.md)
- [ ] Code review approved by senior developer
- [ ] Deployment scripts tested in staging environment
.cursor/workspace.json (Workspace Settings)
json
{
  "version": "1.0.0",
  "name": "DeepFace Ray API Enterprise",
  "description": "Enterprise-ready facial recognition API with Ray distributed computing",
  
  "rules": {
    "focusMode": {
      "enabled": true,
      "batchSize": 5,
      "priorityOrder": ["P0", "P1", "P2"],
      "requireDocumentation": true,
      "requireTests": true
    },
    
    "codeQuality": {
      "enforceTypeHints": true,
      "enforceDocstrings": true,
      "maxComplexity": 10,
      "maxLineLength": 88,
      "enforceSecurityChecks": true
    },
    
    "documentation": {
      "autoUpdateChangelog": true,
      "autoUpdateTodoStatus": true,
      "requireCommitMessages": true,
      "enforceConventionalCommits": true
    },
    
    "testing": {
      "minCoverageThreshold": 90,
      "requireUnitTests": true,
      "requireIntegrationTests": true,
      "runSecurityScans": true
    }
  },
  
  "templates": {
    "todoEntry": {
      "format": "- [ ] **{title}** | Estimated: {hours} hours\n  - Dependencies: {dependencies}\n  - Scope: {scope}\n  - Security Impact: {security_impact}"
    },
    
    "changelogEntry": {
      "format": "### {type} - {priority} {category}\n- **{title}**: {description}\n  - {technical_details}\n  - **Performance**: {performance_impact}\n  - **Security**: {security_impact}"
    },
    
    "commitMessage": {
      "format": "{type}({scope}): {description}\n\n{body}\n\nCloses: {todo_reference}\n{impact_notes}"
    }
  },
  
  "automations": {
    "onTodoComplete": [
      "updateTodoStatus",
      "addChangelogEntry", 
      "runTests",
      "updateDocumentation"
    ],
    
    "onCommit": [
      "validateCommitMessage",
      "runSecurityScan",
      "updateProgressTracking"
    ],
    
    "beforePush": [
      "runFullTestSuite",
      "validateDocumentation",
      "checkCodeQuality"
    ]
  }
}
🎯 Usage Instructions for Development Team
Daily Workflow
Morning: Review TODO.md, select current batch

Implementation: Follow batch rules, implement 3-5 related TODOs

Documentation: Update CHANGELOG.md and TODO.md after each completion

Evening: Commit changes with conventional commit messages

Weekly Reviews
Progress assessment against enterprise roadmap

Security review of implemented features

Performance benchmark validation

Documentation completeness check

Quality Gates
No TODO marked complete without comprehensive tests

All security features must pass penetration testing

Performance must meet enterprise SLAs



Documentation must be production-ready

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codesapienbe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/codesapienbe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
