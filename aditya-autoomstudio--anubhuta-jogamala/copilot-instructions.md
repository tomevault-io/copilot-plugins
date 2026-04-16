## anubhuta-jogamala

> description: High-level project overview and rule index

---
description: High-level project overview and rule index
alwaysApply: true
tags: [project, overview, index]
---
- Provide a concise **project overview** that outlines objectives and scope.
- Use clear, action-oriented language to guide both the AI and developers.
- Link to other rule components for detailed guidance:
  - [context.mdc](mdc:context.mdc) – project context and AI usage rules:contentReference[oaicite:0]{index=0}.
  - [stakeholders.mdc](mdc:stakeholders.mdc) – stakeholders, value, goals, and use case rules:contentReference[oaicite:1]{index=1}.
  - [architecture.mdc](mdc:architecture.mdc) – architecture, design and documentation standards:contentReference[oaicite:2]{index=2}.
- Use internal markdown links (`[file.mdc](mdc:file.mdc)`) for navigation and referencing:contentReference[oaicite:3]{index=3}.

- Keep this entry concise: it should *always* apply (set `alwaysApply: true`) to introduce the project rules to the AI.

# Project Initialization Rules

## Overview
This document defines the standard procedures and requirements for initializing new projects and components within the framework.

## Tags
- setup
- initialization
- project-structure
- bootstrap

## Rule Type: Mandatory

## Project Setup

### 1. Repository Structure
```
project/
├── src/                    # Source code
│   ├── components/         # React components
│   ├── hooks/             # Custom React hooks
│   ├── utils/             # Utility functions
│   ├── services/          # API services
│   ├── types/             # TypeScript types
│   └── styles/            # Global styles
├── tests/                 # Test files
├── docs/                  # Documentation
├── scripts/               # Build and utility scripts
└── config/                # Configuration files
```

### 2. Required Configuration Files
- `package.json` - Project dependencies and scripts
- `tsconfig.json` - TypeScript configuration
- `.eslintrc.js` - ESLint rules
- `.prettierrc` - Code formatting rules
- `jest.config.js` - Testing configuration
- `.env.example` - Environment variables template
- `README.md` - Project documentation

### 3. Dependencies
- Core dependencies must be pinned to specific versions
- Development dependencies should use caret (^) for minor updates
- Security-critical dependencies must be audited regularly

## Component Initialization

### 1. Component Structure
```typescript
// Component template
import React from 'react';
import styles from './Component.module.css';

interface IComponentProps {
  // Props definition
}

export const Component: React.FC<IComponentProps> = ({ prop1, prop2 }) => {
  // Component logic
  return (
    <div className={styles.container}>
      {/* Component JSX */}
    </div>
  );
};

export default Component;
```

### 2. Required Files
- Component file (`.tsx`)
- Style module (`.module.css`)
- Test file (`.spec.tsx`)
- Index file (`.ts`)

## Service Initialization

### 1. API Service Structure
```typescript
// Service template
import { AxiosInstance } from 'axios';

export class Service {
  private client: AxiosInstance;

  constructor(client: AxiosInstance) {
    this.client = client;
  }

  // Service methods
}
```

### 2. Required Files
- Service class file (`.ts`)
- Service interface file (`.ts`)
- Test file (`.spec.ts`)

## Testing Setup

### 1. Test Structure
```typescript
// Test template
import { render, screen } from '@testing-library/react';
import { Component } from './Component';

describe('Component', () => {
  it('should render correctly', () => {
    // Test implementation
  });
});
```

### 2. Required Test Files
- Unit tests for utilities
- Component tests for UI
- Integration tests for services
- E2E tests for critical flows

## Documentation Requirements

### 1. README Structure
```markdown
# Project Name

## Overview
Brief description of the project

## Setup
Installation and setup instructions

## Usage
How to use the project

## Development
Development guidelines

## Testing
Testing instructions

## Contributing
Contribution guidelines
```

### 2. Required Documentation
- API documentation
- Component documentation
- Setup instructions
- Development guidelines

## Examples

### ✅ Good Practice
```typescript
// Proper component initialization
import React from 'react';
import styles from './UserProfile.module.css';

interface IUserProfileProps {
  userId: string;
  displayName: string;
}

export const UserProfile: React.FC<IUserProfileProps> = ({
  userId,
  displayName,
}) => {
  return (
    <div className={styles.container}>
      <h1>{displayName}</h1>
    </div>
  );
};

export default UserProfile;
```

### ❌ Bad Practice
```typescript
// Improper component initialization
import React from 'react';

const UserProfile = ({ id, name }) => {
  return (
    <div>
      <h1>{name}</h1>
    </div>
  );
};

export default UserProfile;
```

## Related Rules
- [Code Conventions](dev/conventions.mdc)
- [Testing Standards](dev/testing-standards.mdc)
- [Architecture Standards](architecture.mdc)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aditya-autoomstudio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
