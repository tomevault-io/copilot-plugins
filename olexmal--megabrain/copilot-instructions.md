## megabrain

> Standard operating procedure for documenting and testing new code


# SOP: Documenting and Testing New Code

## When adding or changing code

### Documentation
1. **Java:** Add Javadoc to public classes and public methods (`@param`, `@return`, `@throws`). Document Mutiny return types (e.g. "Returns Uni that emits...").
2. **TypeScript:** Add JSDoc for public service methods and component inputs/outputs. Document Observable/signal semantics.
3. **User stories:** Update task status and acceptance criteria in the feature file; add technical notes for non-obvious decisions.
4. **Copyright:** Ensure new files have the project copyright header (see copyright-headers.mdc).

### Testing
1. **Backend:** Write unit tests for new/affected behavior; use JUnit 5 and Mockito. Mock external services; use `Uni.createFrom().item(...)` and `.await().indefinitely()` in tests. No reflection in unit tests. Target >80% coverage for new code.
2. **Frontend:** Write Jest spec for new components and services; use TestBed and mocks for HTTP/SSE. Test success and error paths.
3. **Integration:** For new REST or SSE endpoints, add or extend integration tests in `src/it/java` with `@QuarkusIntegrationTest` and base URL including `/api/v1`.
4. Run backend tests: `./mvnw test`; frontend: `npm run test`. Fix any coverage or lint regressions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olexmal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/olexmal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
