## travelcheck

> - **NEVER** mark a feature as "complete" based solely on code existence

# TravelCheck Development Rules

## Core Development Principles

### 1. Functional Validation Over Code Presence
- **NEVER** mark a feature as "complete" based solely on code existence
- **ALWAYS** test actual functionality before claiming completion
- **VALIDATE** that code works as intended, not just that it compiles
- **TEST** critical user flows end-to-end before marking complete

### 2. Architecture Consistency Validation
- **VERIFY** deployment pipeline matches claimed architecture
- **REMOVE** conflicting or duplicate implementations
- **TEST** actual deployment process, not just code structure
- **VALIDATE** all services work together as intended

### 3. Production Readiness Gates
Before marking ANY feature as production-ready:
- ✅ Code exists AND functions correctly
- ✅ Security requirements are met and tested
- ✅ Production deployment has been tested
- ✅ End-to-end user flow has been validated
- ✅ Error handling has been tested
- ✅ Performance requirements are met
- ✅ Monitoring and alerting are configured

### 4. Honest Progress Reporting
- **USE** conservative completion estimates until full validation
- **AVOID** claiming "100% complete" without thorough testing
- **PREFER** "Implementation Complete - Testing Required" over false completion claims
- **DOCUMENT** known issues and limitations clearly

## Implementation Validation Checklist

For each feature implementation:
- [ ] Code implemented and compiles
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Security requirements met and tested
- [ ] Production deployment tested
- [ ] User acceptance testing completed
- [ ] Error scenarios handled and tested
- [ ] Performance requirements met
- [ ] End-to-end user flow validated

## Architecture Review Process

Before major architecture changes:
- [ ] Document current state clearly
- [ ] Plan migration steps in detail
- [ ] Test each migration step
- [ ] Validate final state matches plan
- [ ] Update deployment pipeline accordingly
- [ ] Remove old implementations completely
- [ ] Test new architecture end-to-end

## Security & Production Standards

### OAuth Implementation
- **ALWAYS** test OAuth flows with real providers before marking complete
- **VALIDATE** token storage security (encrypted, not plaintext)
- **TEST** refresh token lifecycle and error handling
- **VERIFY** proper scopes and permissions

### Document AI & External APIs
- **NEVER** use hardcoded values in production code
- **ALWAYS** use environment-driven configuration
- **VALIDATE** API endpoints and authentication
- **TEST** error handling for API failures

### File Operations
- **TEST** file uploads, downloads, and transformations
- **VALIDATE** data format compatibility between frontend/backend
- **VERIFY** file size limits and type restrictions
- **TEST** error scenarios (corrupted files, network failures)

### Database & Storage
- **VALIDATE** data persistence and retrieval
- **TEST** security rules with real data
- **VERIFY** backup and recovery procedures
- **CHECK** data consistency across operations

## Error Handling & Monitoring

### Error Boundaries
- **IMPLEMENT** comprehensive error boundaries
- **TEST** error scenarios and recovery
- **VALIDATE** error reporting and logging
- **VERIFY** user-friendly error messages

### Monitoring & Analytics
- **CONFIGURE** proper monitoring for all critical operations
- **TEST** analytics event tracking
- **VALIDATE** error reporting and alerting
- **VERIFY** performance monitoring

## Code Quality Standards

### Import/Export Consistency
- **VERIFY** all imports match actual exports
- **TEST** runtime behavior, not just compilation
- **VALIDATE** component rendering and functionality
- **CHECK** for unused dependencies and dead code

### Runtime Compatibility
- **VALIDATE** Node.js version compatibility
- **TEST** deployment on target environment
- **VERIFY** all dependencies are compatible
- **CHECK** for runtime errors and warnings

## Testing Requirements

### Unit Testing
- **WRITE** tests for all critical functions
- **TEST** error scenarios and edge cases
- **VALIDATE** input validation and sanitization
- **VERIFY** expected outputs and behaviors

### Integration Testing
- **TEST** service integrations end-to-end
- **VALIDATE** data flow between components
- **VERIFY** error handling across services
- **CHECK** performance under load

### User Acceptance Testing
- **TEST** complete user journeys
- **VALIDATE** user interface functionality
- **VERIFY** accessibility and usability
- **CHECK** cross-browser compatibility

## Deployment & DevOps

### Deployment Pipeline
- **VALIDATE** build process works correctly
- **TEST** deployment to staging environment
- **VERIFY** all environment variables are configured
- **CHECK** rollback procedures work

### Environment Configuration
- **DOCUMENT** all required environment variables
- **VALIDATE** configuration in all environments
- **TEST** configuration changes
- **VERIFY** secrets management

## Communication & Documentation

### Progress Reporting
- **BE HONEST** about implementation status
- **DOCUMENT** known issues and limitations
- **PROVIDE** realistic timelines and estimates
- **COMMUNICATE** blockers and dependencies clearly

### Code Documentation
- **DOCUMENT** complex logic and business rules
- **EXPLAIN** architectural decisions
- **PROVIDE** setup and deployment instructions
- **MAINTAIN** up-to-date README and guides

## Quality Assurance

### Code Review Process
- **REVIEW** for functional correctness, not just style
- **TEST** changes in isolation
- **VALIDATE** security implications
- **VERIFY** performance impact

### Continuous Improvement
- **LEARN** from mistakes and failures
- **IMPROVE** processes based on feedback
- **UPDATE** standards and practices
- **SHARE** knowledge and lessons learned

## Emergency Procedures

### When Issues Are Found
- **STOP** claiming completion until issues are resolved
- **DOCUMENT** the problem clearly
- **PRIORITIZE** fixes based on impact
- **COMMUNICATE** status updates regularly

### Production Issues
- **RESPOND** quickly to critical issues
- **DOCUMENT** root cause analysis
- **IMPLEMENT** fixes with proper testing
- **PREVENT** similar issues in the future

---

## Remember: Code presence ≠ functional completion

Always validate that code works as intended before claiming completion. Better to be honest about progress than to create false expectations that lead to production issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eprasad7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
