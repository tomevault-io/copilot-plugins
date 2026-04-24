## pocket-inventory

> Security base guidelines

---
description: Security base guidelines
globs: 
alwaysApply: false
---
# Critical Security Guidelines
- NEVER hardcode secrets, credentials, tokens, or keys in code
- Use environment variables or secure vaults for sensitive data
- Implement proper authentication and authorization
- Validate all user inputs

# Sensitive Data Handling
- Never log sensitive information (passwords, tokens, PII)
- Encrypt sensitive data at rest and in transit
- Use secure protocols (HTTPS, WSS, etc.)
- Implement proper data retention policies

# Authentication & Authorization
- Use industry-standard authentication methods
- Implement proper session management
- Follow principle of least privilege
- Validate permissions on every protected operation

# Input Validation
- Validate all inputs on both client and server
- Use parameterized queries for database operations
- Sanitize user inputs for display (prevent XSS)
- Implement proper error handling without exposing sensitive details

# Common Vulnerabilities
- Prevent SQL injection through parameterized queries
- Avoid XSS by sanitizing outputs and using CSP
- Prevent CSRF with proper tokens
- Implement rate limiting to prevent brute force attacks

# Infrastructure Security
- Configure proper CORS policies
- Implement security headers
- Use secure cookie settings
- Keep dependencies updated to avoid known vulnerabilities

# Security Testing
- Perform regular security code reviews
- Implement automated security scanning
- Conduct penetration testing
- Monitor for unusual activity

# Agent Behavior
- Scan for security issues first before making changes
- Review data handling for sensitive information
- Check authentication and authorization implementation
- Identify potential security vulnerabilities proactively 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TimLeitch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
