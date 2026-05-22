## tug

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TUG MyFuel is a comprehensive enterprise fuel card management system built with a microservices architecture. The system includes:

- **Backend Services**: 5 NestJS microservices (Auth, Card, Organization, Transaction, API Gateway)
- **Frontend Applications**: 3 React applications (Admin Portal, Customer Portal, POS Terminal)
- **Architecture**: HTTP-based microservices with PostgreSQL databases

## Development Commands

- **Start All Services**: `docker compose up -d` - Starts all services with Docker
- **Build Services**: `npm run build` - Builds all TypeScript services
- **Test Services**: `npm run test` - Runs all service tests
- **Development Mode**: Individual services can be started with `npm run start:dev`

## Project Structure

```
apps/
├── backend/
│   ├── auth-service/          # Port 3002 - Authentication
│   ├── card-service/          # Port 3003 - Card Management  
│   ├── organization-service/  # Port 3004 - Organizations
│   └── transaction-service/   # Port 3005 - Transactions
├── api-gateway/               # Port 3001 - API Gateway
└── frontend/
    ├── admin-portal/          # Port 3100 - Admin Dashboard
    ├── customer-portal/       # Port 3200 - Customer Interface
    └── pos-terminal/          # Port 3050 - POS Terminal
```

## Technology Stack

- **Backend**: NestJS 11.x, TypeScript 5.7.x, PostgreSQL, TypeORM
- **Frontend**: React 18.x, TypeScript, Vite, TailwindCSS
- **Infrastructure**: Docker, Docker Compose

## Important Notes

- All services use TypeScript with strict mode
- Database-per-service pattern with PostgreSQL
- HTTP-based inter-service communication (no gRPC)
- Comprehensive test coverage with Jest
- Development tools organized in `dev-tools/` directory

---
> Source: [omidrahmati2000/TUG](https://github.com/omidrahmati2000/TUG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
