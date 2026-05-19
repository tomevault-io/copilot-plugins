## answer42

> Answer42 is an AI-powered academic research platform: **Java 21 · Spring Boot 3.4.5 · Vaadin 24.7.3 · PostgreSQL (answer42 schema) · Spring AI · Spring Batch**.

# GitHub Copilot Workspace Instructions — Answer42

Answer42 is an AI-powered academic research platform: **Java 21 · Spring Boot 3.4.5 · Vaadin 24.7.3 · PostgreSQL (answer42 schema) · Spring AI · Spring Batch**.

---

## Non-Negotiable Rules

| Rule                      | Detail                                                               |
| ------------------------- | -------------------------------------------------------------------- |
| **No placeholder code**   | Every implementation must be complete and production-ready           |
| **No deprecated APIs**    | Use only current Spring Boot 3.4.5 / Vaadin 24.7.3 patterns          |
| **LoggingUtil always**    | Never use `java.util.logging` or `System.out` — always `LoggingUtil` |
| **Files ≤ 300 lines**     | Split large classes into utility helpers or sub-components           |
| **No inline styles**      | All CSS goes in the Vaadin theme; use external CSS classes           |
| **Routes in UIConstants** | Every Vaadin route string must be defined in `UIConstants.java`      |

---

## Architecture

### Layers (top → bottom)

1. **Vaadin UI** — views extend `Div implements BeforeEnterObserver`; add children directly to `this`
2. **Service** — `@Service` beans, `@Transactional` on write methods, `@Transactional(readOnly=true)` on reads
3. **Agent** — all AI agents extend `AbstractConfigurableAgent`; 9 pipeline agents + 9 Ollama fallback agents
4. **Repository** — Spring Data JPA; UUID PKs; `answer42` schema
5. **Database** — PostgreSQL with JSONB; managed via Supabase migrations

### Key Source Locations

```
src/main/java/com/samjdtechnologies/answer42/
  config/           Spring configuration (AIConfig, SecurityConfig, ThreadConfig…)
  model/db/         JPA entities — UUID IDs, answer42 schema
  repository/       Spring Data JPA repositories
  service/          Business services
  service/agent/    9 pipeline agents + 9 fallback agents (AbstractConfigurableAgent)
  service/discovery/ Multi-source paper discovery (Crossref, Semantic Scholar, Perplexity)
  batch/tasklets/   Spring Batch tasklet implementations
  ui/views/         Vaadin views
  ui/components/    Reusable Vaadin components
  util/             LoggingUtil and other helpers
```

### Multi-Agent Pipeline (Spring Batch)

Sequential 9-step pipeline launched on paper upload:

| Step | Agent                 | Provider            |
| ---- | --------------------- | ------------------- |
| 1    | PaperProcessor        | OpenAI GPT-4        |
| 2    | MetadataEnhancement   | OpenAI GPT-4        |
| 3    | ContentSummarizer     | Anthropic Claude    |
| 4    | ConceptExplainer      | OpenAI GPT-4        |
| 5    | QualityChecker        | Anthropic Claude    |
| 6    | CitationFormatter     | OpenAI GPT-4        |
| 7    | RelatedPaperDiscovery | Claude + Perplexity |
| 8    | CitationVerifier      | OpenAI GPT-4        |
| 9    | PerplexityResearch    | Perplexity API      |

All agents have Ollama (llama3.1:8b) fallback via `FallbackAgentFactory`.

---

## Coding Standards

### Java / Spring

- **4-space indentation**, no tabs, 120-char line limit
- Constructor injection preferred over field injection
- `@Transactional` on all service write methods; `(readOnly = true)` on queries
- `@JdbcTypeCode(SqlTypes.JSON)` for JSONB columns; `List<String>` for JSON arrays
- Custom exceptions with meaningful context messages
- JavaDoc on all public methods with `@param` / `@return`

### Imports order

1. `java.*` / `javax.*`
2. Third-party libraries
3. Project (`com.samjdtechnologies.*`)

### Vaadin UI

- Views: `public class MyView extends Div implements BeforeEnterObserver`
- Add child components with `add(component)` on `this` — never create a wrapper `Div`
- All route constants in `UIConstants.ROUTE_*`
- Use Lumo design-system CSS variables for colours/spacing
- Dark mode via existing theme; no hard-coded colour values

### Logging

```java
// Always use LoggingUtil — never Logger directly in business code
LoggingUtil.info(getClass(), "methodName", "Processing paper id={}", paperId);
LoggingUtil.error(getClass(), "methodName", "Failed for id={}", paperId, ex);
```

---

## Build & Test Commands

```bash
./mvnw clean install               # Full build
./mvnw spring-boot:run             # Dev server (hot-reload)
./mvnw test                        # All tests
./mvnw test -Dtest=Foo#bar         # Single test
./mvnw clean install -Pproduction  # Production build
./mvnw checkstyle:check            # Code style
./mvnw pmd:check                   # Static analysis
./mvnw spotbugs:check              # Bug detection
./mvnw clean verify                # All quality gates
```

---

## Database

- **Schema**: `answer42`
- **Migrations**: Supabase (`supabase/migrations/`); apply with `supabase db push`
- **Entities**: UUID PKs (`@GeneratedValue(strategy = GenerationType.UUID)`)
- **JSONB**: Use `@JdbcTypeCode(SqlTypes.JSON)` with `JsonNode` or `List<String>`
- Use Supabase MCP to inspect schema before writing queries or entities

---

## External Providers

| Provider         | Model                        | Use                              |
| ---------------- | ---------------------------- | -------------------------------- |
| OpenAI           | gpt-4o (prod: gpt-3.5-turbo) | Pipeline steps 1,2,4,6,8         |
| Anthropic        | claude-3-7-sonnet-latest     | Pipeline steps 3,5,7             |
| Perplexity       | sonar-pro                    | Pipeline step 9, discovery       |
| Ollama           | llama3.1:8b                  | Local fallback for all 9 agents  |
| Crossref         | —                            | Citation/bibliographic discovery |
| Semantic Scholar | —                            | Semantic similarity discovery    |

---

## Security Checklist

- Never log API keys, passwords, or paper content
- Validate all user inputs at controller/view layer
- JWT tokens for stateless auth via `SecurityConfig`
- Encode passwords before storing (BCrypt)
- Parameterised queries only — no string-concatenated SQL

---
> Source: [samjd-zz/answer42](https://github.com/samjd-zz/answer42) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
