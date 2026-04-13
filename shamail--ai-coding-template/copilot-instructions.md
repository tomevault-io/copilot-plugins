## ai-coding-template

> Defines performance standards and optimization practices


# Application Performance Requirements

This document outlines requirements for application performance to ensure optimal user experience and system efficiency. All development **MUST** strive to meet these.

## 1. Frontend Performance

### 1.1. Efficient Component Rendering (React)
*   Components **MUST** be optimized to prevent unnecessary re-renders (e.g., `React.memo`, `useMemo`, `useCallback`, efficient state updates).
*   Large lists/datasets **MUST** be rendered performantly (e.g., virtualization/windowing, stable keys for list items).
*   Avoid expensive computations directly within the render path; memoize or move to effects if possible.
*   Avoid expensive computations directly within the render path; memoize or move to effects if possible.

*   Static assets (images, fonts, videos) **MUST** be optimized for web delivery:
    *   Use appropriate formats (e.g., WebP for images, WOFF2 for fonts).
    *   Compress assets without significant quality loss.
    *   Implement responsive images (`<picture>` element or `srcset` attribute).
*   Build processes **SHOULD** automatically minify CSS and JavaScript.
*   Lazy loading for offscreen images and non-critical components **MUST** be implemented.
    *   Use appropriate formats (e.g., WebP for images, WOFF2 for fonts).
    *   Compress assets without significant quality loss.
*   Applications **MUST** utilize code-splitting (e.g., route-based splitting) to reduce the initial JavaScript payload and improve Time to Interactive (TTI).
*   Minimize blocking JavaScript; defer or load asynchronously where possible.
*   Analyze bundle sizes regularly and remove unused code or dependencies.
*   Consider prefetching/preloading critical resources for key user flows.

### 1.3. Code Delivery and Execution
*   Applications **MUST** utilize code-splitting (e.g., route-based splitting) to reduce the initial JavaScript payload and improve Time to Interactive (TTI).
*   Minimize blocking JavaScript; defer or load asynchronously where possible.
*   Analyze bundle sizes regularly and remove unused code or dependencies.
*   Consider prefetching/preloading critical resources for key user flows.

## 2. Backend and Data Performance

### 2.1. API Efficiency (Hono)
*   API endpoints **MUST** respond efficiently with minimal latency. Target p95/p99 latencies based on service criticality.
*   Data fetching from APIs **MUST** minimize redundant data transfer (e.g., use GraphQL if appropriate, pagination, filtering, return only necessary fields).
*   API responses **SHOULD** be compressed (e.g., Gzip, Brotli).
*   Optimize JSON serialization/deserialization if it becomes a bottleneck.
*   Optimize JSON serialization/deserialization if it becomes a bottleneck.
### 2.2. Database Optimization (Drizzle ORM & PostgreSQL)
*   Database queries **MUST** be efficient and optimized. Analyze slow queries (e.g., using `EXPLAIN ANALYZE`).
*   Appropriate database indexing (e.g., on foreign keys, frequently filtered columns) **MUST** be implemented.
*   Avoid N+1 query problems; use eager loading or batching.
*   Connection pooling **MUST** be used effectively.
*   For write-heavy workloads, consider optimizing transaction handling.
*   Avoid N+1 query problems; use eager loading or batching.
*   Connection pooling **MUST** be used effectively.
*   For write-heavy workloads, consider optimizing transaction handling.

## 3. General Performance Practices

### 3.1. Performance Monitoring
*   Applications **SHOULD** integrate Application Performance Monitoring (APM) tools (e.g., Sentry, Datadog, New Relic) to track key metrics (response times, error rates, resource usage).
*   Frontend **SHOULD** monitor Core Web Vitals.
*   Establish performance baselines and set alerts for regressions.
*   Frontend **SHOULD** monitor Core Web Vitals.
*   Establish performance baselines and set alerts for regressions.
*   Performance testing (load, stress, soak) **SHOULD** be part of the development lifecycle for critical user flows and APIs.
*   Identify and test against expected peak loads.
*   Automate performance tests in CI/CD pipelines where feasible.
### 3.2. Performance Testing
*   Performance testing (load, stress, soak) **SHOULD** be part of the development lifecycle for critical user flows and APIs.
*   Identify and test against expected peak loads.
*   Automate performance tests in CI/CD pipelines where feasible.

## 4. AI-Assisted Performance Enhancement

AI can help identify and address performance issues. Refer to `prompt_engineering_guidelines.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shamail) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
