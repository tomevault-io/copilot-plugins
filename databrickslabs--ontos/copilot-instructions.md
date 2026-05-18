## 06-performance-optimization

> - **Asynchronous Operations**: Use `async/await` for SDK calls and potentially database operations if using an async driver.


### Performance Optimization

**Backend**

- **Asynchronous Operations**: Use `async/await` for SDK calls and potentially database operations if using an async driver.
- **Database Query Optimization**: Use efficient SQLAlchemy queries (e.g., `selectinload` for eager loading relationships in repositories). Add database indexes where needed (`db_models`).
- **Caching**: Consider caching for frequently accessed, rarely changing data (e.g., settings, roles) if performance becomes an issue.

**Frontend**

- **Component Memoization**: Use `React.memo`, `useMemo`, `useCallback`.
- **Bundle Size**: Monitor bundle size and use code splitting/lazy loading if necessary.
- **Efficient Data Fetching**: Fetch only necessary data. Use libraries like TanStack Query for caching/staleness management if needed.

---
> Source: [databrickslabs/ontos](https://github.com/databrickslabs/ontos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
