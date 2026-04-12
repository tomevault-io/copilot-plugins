## hackathon

> Systematic approach to debugging and fixing issues


# Debugging & Bug Fix Guidelines

## Initial Investigation Protocol

### 1. Comprehensive System Analysis
- **STOP and assess before making any changes**
- Examine the complete error context (stack traces, logs, console output)
- List all files potentially related to the issue
- Review recent changes that might have introduced the bug
- Check for similar issues in project history or documentation

### 2. Root Cause Analysis
- Reproduce the issue consistently with minimal steps
- Identify the exact failure point and conditions
- Trace the data flow and execution path
- Check for environment-specific factors (dependencies, configuration)
- Verify assumptions about expected vs. actual behavior

## Debugging Methodology

### Evidence-Based Investigation
- Gather all available error information (logs, network requests, console errors)
- Check multiple browsers/environments if applicable
- Test with different data sets and user scenarios
- Document what works vs. what doesn't work
- Look for patterns in failure conditions

### Systematic Troubleshooting
1. **Start with the obvious**: Check syntax, imports, and basic configuration
2. **Isolate the problem**: Create minimal reproduction cases
3. **Work backwards**: Trace from the error point to the source
4. **Check dependencies**: Verify versions, configurations, and compatibility
5. **Test incrementally**: Make small changes and test after each one

## Common Issue Categories

### Runtime Errors
- Check for undefined variables or null pointer exceptions
- Verify function parameters and return types
- Look for scope and closure issues
- Check async/await and Promise handling
- Verify event handlers and lifecycle methods

### Logic Errors
- Trace through conditional statements and loops
- Verify mathematical operations and comparisons
- Check data transformations and mappings
- Look for off-by-one errors and boundary conditions
- Verify state updates and side effects

### Integration Issues
- Check API endpoints and data formats
- Verify authentication and authorization
- Look for CORS and network-related problems
- Check database queries and transactions
- Verify third-party service integrations

### Performance Issues
- Profile code execution and identify bottlenecks
- Check for memory leaks and resource usage
- Look for N+1 queries and inefficient algorithms
- Verify caching strategies and implementations
- Check bundle sizes and loading performance

## Fix Implementation Strategy

### Safe Remediation Approach
1. **Create a backup**: Save current state before making changes
2. **Make minimal changes**: Address the root cause with smallest possible fix
3. **Test thoroughly**: Verify fix works and doesn't break other functionality
4. **Add preventive measures**: Include tests to prevent regression
5. **Document the solution**: Record the issue and resolution for future reference

### Fix Validation Process
- Reproduce the original issue to confirm the fix
- Run comprehensive tests (unit, integration, e2e)
- Test edge cases and error scenarios
- Verify performance impact is acceptable
- Check for side effects in related functionality

## Error Prevention Measures

### Immediate Actions
- Add proper error handling around the fixed code
- Include input validation and boundary checks
- Add logging for better future debugging
- Implement graceful degradation where appropriate
- Add monitoring and alerting if applicable

### Long-term Improvements
- Write tests that would have caught this bug
- Update documentation to prevent similar issues
- Consider architectural improvements to avoid the issue class
- Add code review checkpoints for similar patterns
- Update team knowledge and coding standards

## Documentation Requirements
- Document the issue symptoms and root cause
- Record the investigation process and findings
- Explain the fix implementation and reasoning
- Add prevention measures and future considerations
- Update relevant documentation (README, troubleshooting guides)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ujjwaljain0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ujjwaljain0)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
