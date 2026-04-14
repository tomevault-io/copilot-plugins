## newsaggregator

> The `NewsAggregator` project is a background service designed to aggregate technical news. It scrapes headlines from various tech-oriented sources based on predefined preferences and compiles the gathered data into a `Markdown` file.

# Project Purpose

The `NewsAggregator` project is a background service designed to aggregate technical news. It scrapes headlines from various tech-oriented sources based on predefined preferences and compiles the gathered data into a `Markdown` file.

This application is strictly a background service and does not function as an API, nor does it feature a frontend interface. It operates by generating and saving Markdown files, which include direct links to the source articles. The scraping service executes daily at 08:00. Each file is stored in the `.\Documents\News\` directory, utilizing the `MM-DD-YYYY.md` naming convention.

# Technology Stack

The project leverages **.NET** and **C# 13** as its core technologies. The testing suite utilizes **xUnit** alongside **Moq** to simulate various test scenarios. Web scraping is facilitated by the **HtmlAgilityPack** library, while **MarkdownLog** handles the generation of the `.md` files.

# Architecture and Directory Structure

The project adheres strictly to **Domain-Driven Design (DDD)** principles, ensuring rich domain models rather than anemic entities. All source code is located within the `src/` directory.

* **`NewsAggregator.Domain`**: The core of the application containing business logic.
* `Interfaces/`: Defines essential architectural contracts.
* `Entities/`: Contains rich domain entities.
* `ValueObjects/` & `Exceptions/`: Houses domain-specific values and custom exceptions.


* **`NewsAggregator.Application`**: Orchestrates domain logic and contains `UseCases` and `DTOs`.
* **`NewsAggregator.Infrastructure`**: Manages external concerns, such as the `Scraping/` logic and `FileSystem/` operations for Markdown persistence.
* **`NewsAggregator.Worker`**: The entry point for background execution.

The `tests/` directory mirrors the `src/` structure. Each test project corresponds to its respective implementation project (e.g., `NewsAggregator.Domain.Tests/`).

```plaintext
NewsAggregator.sln
├── src/
│   ├── NewsAggregator.Domain/          # The application core
│   │   ├── Entities/                   # Rich domain entities (no anemic models)
│   │   ├── Exceptions/                 # Domain-specific exceptions
│   │   ├── Interfaces/                 # The REAL NECESSARY contracts
│   │   └── ValueObjects/               # Domain value objects
│   │
│   ├── NewsAggregator.Application/     # Domain logic orchestration
│   │   ├── DTOs/                       # Data Transfer Objects (if necessary)
│   │   └── UseCases/                   # Application use cases
│   │
│   ├── NewsAggregator.Infrastructure/  # Where it actually works (external details)
│   │   ├── FileSystem/                 # Logic to write markdown files
│   │   └── Scraping/                   # Web scraping implementations
│   │
│   └── NewsAggregator.Worker/          # Entry point
│       └── Workers/                    # Background workers
│
└── tests/                              # Unit and integration tests
    ├── NewsAggregator.Application.Tests/
    ├── NewsAggregator.Domain.Tests/
    └── NewsAggregator.Infrastructure.Tests/
```

# Engineering Principles

We strictly follow **Test-Driven Development (TDD)**: no production code is written until a corresponding test has been created and failed. All services must utilize **Dependency Injection** via constructor interfaces. To maintain isolation, unit tests are prohibited from making actual HTTP requests; all external dependencies must be mocked using **Moq**.

# Business Logic and Rules

All domain logic resides within the `NewsAggregator.Domain` layer:

* **Article Integrity**: An article cannot be instantiated without a title, a valid URL, and a specified source. Failure to meet these criteria triggers an `InvalidArticleException`.
* **URL Validation**: Article URLs are encapsulated within a **ValueObject** to ensure they are valid `http/https` links.
* **Immutability**: Once an article is created, it remains immutable. All attributes use private setters to prevent post-instantiation modifications.
* **Slugification**: All article titles must be processed by a domain service (e.g., `ISlugifierService`) to generate clean, SEO-friendly strings. The logic must convert text to lowercase, remove all accents (diacritics), strip special characters, and replace spaces with hyphens.

## Domain Services and Use Cases

* **Deduplication**: The system prevents duplicate entries by verifying that no two articles share the same URL or title from the same source.
* **Constraint**: The application retrieves only the five most recent news items per source.
* **Organization**: Articles are grouped according to their respective portals.

## Application Resilience

* **Fault Tolerance**: If a scraping operation fails for a specific portal, the error is logged, and the service proceeds to the next source without interruption.
* **Scheduling**: The application utilizes **Quartz.NET** to reliably manage background jobs. A Quartz `IJob` must be configured with a Cron trigger to execute the aggregation process precisely every day at 08:00 AM.

# Output Format (Markdown Template)

The generated `.md` files (saved in `.\Documents\News\MM-DD-YYYY.md`) must strictly follow this structural template, organized by source:

```markdown
# Tech News Aggregator - [MM-DD-YYYY]

## [Source Name 1]
* [[Article Title 1]](Article Valid URL)
* [[Article Title 2]](Article Valid URL)

## [Source Name 2]
* [[Article Title 1]](Article Valid URL)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/babisque)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/babisque)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
