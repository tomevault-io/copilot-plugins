## botcartsaas

> This file defines the coding standards and practices to be followed when working on the Botcart SaaS platform.


This file defines the coding standards and practices to be followed when working on the Botcart SaaS platform.
Reference Architecture

Context Reference: When uncertain about implementation details or architecture decisions, always refer back to the context.md file
Knowledge Gaps: If a request is unclear or lacks necessary details, consult the context.md file before suggesting implementations
Architectural Consistency: Ensure all code changes align with the overall architecture defined in context.md
Feature Boundaries: When working on a specific feature, check context.md to understand how it integrates with other components

Core Principles

Simplicity First: Always prefer simple, readable solutions over complex ones
Clean Architecture: Maintain clear separation of concerns between components
Security Focused: Handle authentication and data with proper security measures
Future-Ready: Write code that can easily accommodate the planned subscription tiers

Development Practices

Avoid Duplication: Before writing new code, check if similar functionality exists
Environment Awareness: Ensure code works across dev, test, and prod environments
Change Scope: Only modify code that's directly related to the requested changes
Pattern Consistency: Don't introduce new patterns or technologies without exhausting existing options
Clean Refactoring: When replacing code, remove old implementations completely

Backend Specifics

Authentication: Keep all OAuth flows in the backend for security
Token Management: Handle all token storage and refresh in the backend
API Design: Use RESTful principles with proper status codes and error handling
Multi-Tenant: Always filter database queries by user_id for proper isolation
Subscription Awareness: Check entitlements before performing tier-limited operations

Context-Driven Development

Refer to Context: When requirements are unclear or ambiguous, always consult context.md first
Architecture Alignment: Ensure implementations match the specified architecture in context.md
Feature Integration: Check context.md to understand how features should interact with each other
Knowledge Gaps: Before asking for clarification, verify if the answer exists in context.md
Implementation Priorities: Use context.md to determine the priority of implementation details

Remember: The Botcart platform handles business communication on behalf of users. Code quality directly impacts user experience and business reputation. Always prioritize security, reliability, and maintainability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronoksheikh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
