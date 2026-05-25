## mkt-gus

> - Project name: MKT-GUS

# MKT-GUS Project Instructions

## Project Context

- Project name: MKT-GUS
- Product type: Monorepo with Spring Boot backend and Next.js frontend
- Primary backend language: Java 21
- Backend framework: Spring Boot 3, Spring Web, Spring Data JPA, WebSocket
- Package manager: Maven Wrapper
- Runtime and platform: JVM, MySQL or compatible relational database

## Architecture

- Backend architecture style: Pragmatic Clean Architecture
- Entry points: REST controllers and WebSocket/barcode integrations
- Main backend domains: customer, employee, coupon, order, product lookup
- External integrations: Mercado Livre product catalog and DroidCam barcode stream
- Data access approach: domain ports with JPA adapters

## Development Workflow

- Follow SDD before implementation: define requirement, constraints, expected behavior, and acceptance outcome.
- Prefer incremental refactors that preserve current API routes and behavior.
- Keep backend business rules inside application and domain layers.

## Backend Standards

- Keep business logic out of controllers and framework-specific classes.
- Model database and external services behind ports so they can be replaced later.
- Preserve current contracts unless a breaking change is explicitly approved.
- Keep files under 300 lines and generated code lines under 150 characters.
- Run relevant Maven tests after backend changes whenever possible.

## Delivery Checklist

- [ ] Requirement and acceptance outcome were identified before coding.
- [ ] Controllers delegate to use cases instead of holding business logic.
- [ ] Database and external services are accessed through ports/adapters.
- [ ] Files stay within the project limits.
- [ ] Relevant tests were run, or the lack of tests was stated.

---
> Source: [JoaoPires3642/MKT-GUS](https://github.com/JoaoPires3642/MKT-GUS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
