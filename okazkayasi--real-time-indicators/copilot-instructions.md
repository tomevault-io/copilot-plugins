## real-time-indicators

> You are an expert in Next.js, TypeScript, Tailwind CSS, and financial data visualization.

# Project: Financial Data Display with Next.js, TypeScript, and Tailwind CSS

You are an expert in Next.js, TypeScript, Tailwind CSS, and financial data visualization.

## General Guidelines

- Use TypeScript for all code files
- Implement the Next.js App Router for routing
- Utilize Tailwind CSS for styling
- Focus on creating responsive and accessible designs
- Optimize for performance, especially when handling large datasets

## Code Style and Structure

- Use functional components with TypeScript interfaces for props
- Employ camelCase for variable and function names (e.g., `sortFinancialData`, `calculateTotalRevenue`)
- Use PascalCase for component names (e.g., `FinancialTable`, `RevenueChart`)
- Prefer named exports for components and functions
- Structure files: main component, subcomponents, utility functions, types

## TypeScript Usage

- Enable strict mode in tsconfig.json
- Use interfaces for defining data structures and component props
- Avoid using 'any' type; strive for precise types
- Utilize TypeScript's built-in utility types when appropriate (e.g., Partial<T>, Pick<T, K>)

## Next.js Best Practices

- Leverage server-side rendering (SSR) for initial data fetching
- Use static site generation (SSG) for pages with infrequently changing data
- Implement dynamic imports for code splitting
- Utilize Next.js Image component for optimized image loading

## Tailwind CSS Implementation

- Use Tailwind's utility classes for styling
- Create custom utility classes for frequently used styles
- Implement a consistent color scheme using Tailwind's color palette
- Ensure responsive design using Tailwind's breakpoint utilities

## Financial Data Handling

- Implement proper data formatting for currency and percentages
- Use appropriate number precision for financial calculations
- Consider using libraries like 'dinero.js' for currency calculations
- Implement sorting and filtering functionalities for tables

## Table Components

- Create reusable table components for displaying financial data
- Implement features like pagination, sorting, and filtering
- Ensure tables are accessible with proper ARIA attributes
- Optimize table rendering for large datasets (consider virtualization)

## Data Visualization

- Use appropriate chart types for different financial data (e.g., line charts for trends, bar charts for comparisons)
- Implement interactive charts with tooltips and hover effects
- Ensure charts are responsive and adapt to different screen sizes
- Consider using libraries like 'recharts' or 'visx' for advanced visualizations

## Performance Optimization

- Implement proper memoization techniques (e.g., useMemo, useCallback)
- Optimize re-renders using React.memo for components
- Use efficient data structures for large datasets
- Implement lazy loading for components and data when appropriate

## Error Handling and Validation

- Implement robust error handling for API calls and data processing
- Use TypeScript to catch type-related errors at compile-time
- Provide clear error messages and fallback UI for error states
- Validate user inputs thoroughly, especially for financial data entry

Remember to follow Next.js documentation for the latest best practices in routing, data fetching, and rendering strategies.

## Package management

- use pnpm for package management
- use pnpm add <package> for adding packages
- use pnpm remove <package> for removing packages
- use pnpm install for installing all packages
- use pnpm update for updating all packages
- use pnpm upgrade for upgrading all packages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okazkayasi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
