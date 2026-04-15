## samudra-paket-app

> - State a concise, aspirational vision solving the core business problem.


# Windsurf Rules for Product Requirements

## 1. Vision & Objectives
- State a concise, aspirational vision solving the core business problem.
- Define measurable objectives: operational efficiency, finance, customer, scalability.
- Set KPIs for operations, finance, customer, and system; review quarterly.

## 2. Structure & Components
- Separate Web, Mobile, Backend, and Database with clear boundaries.
- Organize modules by business function: Auth, Dashboard, Branch, Employee, Operations, Finance, Reporting.
- Map requirements to user personas (Management, Admin, Field Ops, Finance) and prioritize by impact.

## 3. Functional Requirements
- Use SMART, traceable, and acceptance-criteria-based requirements by module.
- Core: RBAC auth with MFA, real-time dashboards, end-to-end logistics, order-to-cash, offline sync.
- Mobile: Offline-first, field-optimized UI, device integration (GPS, camera, biometrics), efficient usage.

## 4. Non-Functional Requirements
- Performance: Web <3s, Mobile <2s, DB <1s, reports <30s, 100+ users.
- Security: Encrypt data in transit/at rest, strong auth, audit logs, 30-min timeout, rate limiting.
- Reliability: 99.5% uptime, daily backup, auto-failover, graceful errors, disaster recovery.
- Compatibility: Latest Chrome/Firefox/Safari/Edge, Android 7+/iOS 12+, print/email integration.
- Scalability: Horizontal scale, growing transactions/branches, optimized DB/caching.
- Usability: Consistent UI, minimal steps, contextual help, WCAG 2.1 AA, mobile usability.

## 5. Integration
- Integrate with maps (routing, tracking), payment gateways, SMS/email, forwarders.
- Secure/authenticated APIs, documented interfaces, error handling, data mapping.

## 6. Implementation
- 4 phases/12 months, each with deliverables & acceptance; stakeholder approval for transitions.
- Success: 95% adoption (3mo), 80% manual reduction, 30% efficiency gain, 20% satisfaction boost, ROI <18mo.

## 7. Documentation
- Standardized requirements: ID, description, criteria, priority, traceability, versioning.
- Document architecture, data models (ERD), APIs (OpenAPI), workflows (sequence diagrams).

## 8. Quality Assurance
- Test coverage: unit, integration, end-to-end, performance, security (vulnerability).
- UAT for all functions, performance validation, security audit, verified data migration.

## 9. Maintenance & Support
- Monitoring/alerting, detailed logs, troubleshooting docs, defined support levels.
- Zero-downtime updates, rollback, backward compatibility, forced mobile updates for critical fixes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammadhilmi007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
