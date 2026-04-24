## coda-doc-scraper

> - Install dependencies: `npm install`

# Coda Doc Scraper Development Guide

## Build and Test Commands
- Install dependencies: `npm install`
- Start development server: `npm start`
- Build for production: `npm run build`
- Run tests: `npm test`
- Run a single test: `npm test -- -t "test name"`

## Code Style Guidelines

### Formatting
- React functional components with arrow function syntax
- 2-space indentation
- Tailwind CSS for styling with utility classes

### Imports
- Group imports by: React, third-party libraries, components, utilities
- Use absolute imports for project modules

### Error Handling
- Wrap API calls in try/catch blocks
- Set error state and display with ErrorDisplay component
- Log errors to console with descriptive messages

### Component Structure
- Component folders should contain related components
- Separate presentational and container components
- Use props destructuring in function parameters

### State Management
- Use React hooks (useState, useEffect)
- Prefer local state with prop drilling to manage data flow
- Use localStorage for persistent configuration

### Naming Conventions
- CamelCase for variables, functions, and components
- PascalCase for component files and React components
- Descriptive function names (handle* for event handlers)

## Data Structure
- Column-centric data organization with row values nested under each column
- Transformation of column IDs to human-readable names in row values
- Simplification of nested objects to show only their names
- Complete example:
```json
{
  "TableName": {
    "ColumnName": {
      "name": "ColumnName",
      "display": true,
      "format": {...},
      "rows": [
        {
          "name": "RowName",
          "createdAt": "2025-03-24T12:34:56.789Z",
          "value": "Cell value"
        }
      ]
    }
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jondallasjr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
