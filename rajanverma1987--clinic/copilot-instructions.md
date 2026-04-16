## clinic

> description: "Base architecture rules."

version: 1

rules:


  - id: architecture
    description: "Base architecture rules."
    patterns:
      - "**/*.js"
      - "**/*.jsx"
    actions:
      - "Every MongoDB collection must include tenantId."
      - "All queries must filter by tenantId."
      - "Use a clean service-layer. API routes should stay thin."
      - "No business logic inside UI components."
      - "All modules must be API-first to support future mobile apps."

  - id: backend-api
    description: "API route rules for Next.js."
    patterns:
      - "app/api/**"
    actions:
      - "Use async/await consistently."
      - "Always validate input using custom validators or JOI/Zod (JS version)."
      - "Never return raw MongoDB errors."
      - "Always wrap responses as { success, data, error }."
      - "Every sensitive route must enforce auth + role checks."
      - "Apply rate limiting for public and auth routes."
      - "No PHI in URL params, logs, or notifications."

  - id: database
    description: "MongoDB standards."
    patterns:
      - "lib/db/**"
      - "models/**"
    actions:
      - "Use Mongoose or native Mongo driver with schemas."
      - "All schemas require tenantId."
      - "Use timestamps: true for all collections."
      - "Store all dates as UTC."
      - "Use indexes for heavy read modules (appointments, billing, inventory)."
      - "Implement soft delete where laws allow."
      - "Use encryption helpers for PHI fields."

  - id: security-compliance
    description: "HIPAA/GDPR compliance enforcement."
    patterns:
      - "**/*.js"
    actions:
      - "Always sanitize input to avoid injection."
      - "Encrypt PHI fields before saving to DB."
      - "Implement audit logs on create/update/delete and sensitive read."
      - "No PHI in logs, SMS, WhatsApp, or email notifications."
      - "Implement consent tracking for patient data processing."
      - "Ensure region-specific templates for prescriptions and invoices."
      - "Use access-control middleware for RBAC and ABAC."

  - id: internationalization
    description: "Rules for global-ready product."
    patterns:
      - "app/**"
      - "components/**"
    actions:
      - "Never hardcode currency, timezone, or locale."
      - "Use tenantSettings for locale, timezone, currency."
      - "Use i18n dictionaries for all UI text."
      - "Prescription and invoice templates must be region-specific."

  - id: notification-system
    description: "Rules for reminders, emails, WhatsApp, SMS."
    patterns:
      - "lib/notifications/**"
    actions:
      - "Use background jobs or cron for sending reminders."
      - "Never include PHI in notifications."
      - "Store logs of delivery attempts in notificationLogs."
      - "Allow clinics to choose communication channels."

  - id: ui-structure
    description: "Frontend rules for Next.js."
    patterns:
      - "app/**"
      - "components/**"
    actions:
      - "Use server components for heavy data pages."
      - "Use client components only where required."
      - "Use React Query or SWR for client-side fetching."
      - "Use reusable form and table components."
      - "Avoid inline logic inside components. Move to hooks/services."

  - id: performance
    description: "Performance & scalability standards."
    patterns:
      - "**/*.js"
    actions:
      - "Cache read-heavy endpoints with Redis or in-memory caching."
      - "Use lean() in MongoDB queries when returning plain objects."
      - "Avoid N+1 queries. Prefer aggregated lookups."
      - "Use pagination for all list endpoints."
      - "Use worker queues for long tasks (reports, reminders)."

  - id: future-proofing
    description: "Rules to keep the system extensible."
    patterns:
      - "**/*.js"
    actions:
      - "Build modules with extension points for labs, radiology, pharmacy."
      - "Structure services so they can be converted to microservices later."
      - "Keep region-specific logic isolated behind strategy interfaces."
      - "All new modules must support multi-clinic and multi-region out of the box."

  - id: documentation-and-quality
    description: "Important docs, enterprise comments, and deep thinking."
    patterns:
      - "**/*.js"
      - "**/*.jsx"
      - "CursorMD/**"
    actions:
      - "Always follow CursorMD/New: read and adhere to all files in CursorMD/New (clinic-complete-specification.md, clinic-dashboard-architecture.mermaid, database-schema.mermaid, realtime-caching-strategy.md) before implementing or changing features, permissions, data models, APIs, or real-time behavior."
      - "Always create or update important MD files in CursorMD for significant features, architecture changes, decisions, and specs; keep them accurate and discoverable."
      - "Comments in code must be proper and enterprise-level: explain why and context, not just what; use JSDoc for public APIs; avoid noise and redundant comments."
      - "Always think deep, not quick: consider edge cases, security, performance, multi-tenant and i18n impact, and long-term maintainability before implementing."


final rule

- id: final-rules
  description: "always compile and fix the error, after fixing, kill the npm run dev process and restart it"
  validatoins Error : in Resoonse always return what is the actual error and what is the expected error and what is the solution to fix the error.
  keep in mind the multi language support when adding new element on the FE.
  Never create unnecessary documentation. Only create or update .MD files in CursorMD when the user explicitly asks for documentation.

  No Typescript code should be written, only JavaScript.
  dont compile the code unless i ask you to compile it.
  dont build the code unless i ask you to build it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajanverma1987) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
