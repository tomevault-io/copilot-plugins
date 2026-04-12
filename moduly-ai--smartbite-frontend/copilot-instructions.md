## smartbite-frontend

> **SmartBite Frontend** is a React-based cash reconciliation system for restaurant management. Built with React 18, Vite, and Tailwind CSS, it provides comprehensive employee/owner authentication and multi-step cash reconciliation workflows.

# SmartBite Frontend - AI Agent Documentation

## Project Overview

**SmartBite Frontend** is a React-based cash reconciliation system for restaurant management. Built with React 18, Vite, and Tailwind CSS, it provides comprehensive employee/owner authentication and multi-step cash reconciliation workflows.

## Current System Status (August 2025)

### ✅ FULLY OPERATIONAL

### Live API Endpoints ✅

## Technical Architecture

### Stack

### Key Services

## Project Structure

```
src/
├── components/layout/           # Layout components
├── features/auth/               # Authentication (LoginScreen.jsx)
├── modules/cash-reconciliation/ # Main reconciliation workflows
│   ├── EmployeeReconciliation.jsx
│   └── OwnerReconciliationReview.jsx
├── services/                    # API integration layer
│   ├── apiClient.js            # HTTP client with Bearer auth
│   ├── authService.js          # Authentication management
│   ├── configService.js        # Configuration API
│   └── reconciliationService.js
├── config/env.js               # Environment configuration
└── App.jsx                     # Main app component
```

## Current Technical State

### Live Demo Credentials

### Configuration API Schema
```javascript
// GET /config/system response:
{
  success: true,
  config: {
    registers: { count: 2, names: ["Register 1", "Register 2"], reserveAmount: 400, enabled: [true, true] },
    business: { name: "SmartBite Restaurant", timezone: "Australia/Sydney", currency: "AUD", taxRate: 0.10 },
    reconciliation: { dailyDeadline: "23:59", varianceTolerance: 5.00, requireManagerApproval: true },
    posTerminals: { count: 4, names: ["Terminal 1-4"], enabled: [true, true, true, false] }
  },
  timestamp: "2025-08-19T..."
}

// PUT /config/system - automatically cleans server fields before submission
```

### Authentication Flow
1. `authService.login(employeeId, pin)` → JWT Bearer token
2. Token auto-included in all API requests via `apiClient.js`
3. Role-based access (employee/owner) enforced server-side
4. Session persistence in localStorage with expiration handling

## Application Features

### 1. Multi-Step Cash Reconciliation

### 2. Dynamic Configuration Management

### 3. Role-Based Authentication

## Development Context

### Service Layer Architecture

### Key Technical Patterns

### Environment Variables

## Deployment


## Common Issues & Solutions

### React Component Patterns

### API Integration Issues

### Configuration Management

### Development Scripts


**For AI Agents**: This system is fully operational with live API integration. Focus on maintaining the existing patterns and architecture when making changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Moduly-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Moduly-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
