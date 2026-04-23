## atlas

> Enterprise application portfolio (EAP) for asset inventory management.

# About the Project

Enterprise application portfolio (EAP) for asset inventory management.
Supports regulatory needs including acts like DORA, European Cyber Resiliency Act (CRA), Cyber Security and CIA compliance.
Whenever you create or modify model always think about compliance with regulatory needs and make suggestions
if something is not compliant (e.g. if the BIA attributes are at the right model / entity).
The target is to deliver comprehensive EA tool that combines EAP, CMDB, Vendor / License / Contract Management for the holistic view.
This is not the ticketing or ITSM tool in the current stage.
The tool should bring the modern ideas that simplify to the maximum the way people could look after the quality of the inventory.

## Server

The server is built with NestJS and Prisma. The database is PostgreSQL.
The main goal for the server is to be model-agnostic and to support dynamic model changes.
No assumptions about the model (like entities, types or relations) should be made. 
Entities, types and relations are dynamic and can be changed at runtime.

## Important Design and Code considerations

* **Security First**: Implement OWASP Top 10 protections, input validation, JWT auth, RBAC, and data encryption
* **TypeScript Excellence**: Strict mode enabled, proper typing, no `any`, comprehensive error handling
* **React Best Practices**: Functional components with hooks, proper state management, memoization, accessibility
* **NestJS Standards**: Dependency injection, guards, interceptors, validation pipes, proper error handling
* **Multi-tenancy Ready**: Tenant isolation at database level, tenant-aware middleware, secure data segregation
* High code quality with ergonomic UX and consistent patterns
* Prefer @/components/ui over custom components, avoid duplication
* Avoid hardcoding model lists - prepare for dynamic changes

## Database Design

* Use PostgreSQL with JSONB for dynamic fields
* Follow the hybrid approach with core columns and JSONB attributes
* Ensure proper indexing on JSONB fields for query performance
* Multi-tenant architecture with tenant_id column for data isolation
* Row Level Security (RLS) policies for tenant data protection

## Maintain the project

* Update main README.md and server/README.md when you make changes to the project
* Update server/prisma/seed.ts when you make changes to the database schema

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pmalirz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
