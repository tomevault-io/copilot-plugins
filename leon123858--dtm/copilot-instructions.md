## dtm

> **Project Name:** Division Trip Money (dtm)

# DTM Project Analysis Report

## 1\. Project Overview

**Project Name:** Division Trip Money (dtm)

**Core Objective:** This project is an expense-splitting application written in Go. It is designed to help members of a group activity (such as a trip) easily calculate the amount each person should pay or receive to balance the accounts.

**Operation Modes:** The project offers two main modes of operation:

  - **Command-Line Interface (CLI):** Used for quickly processing CSV files to perform expense-sharing calculations.
  - **Web Service:** Provides a more feature-rich GraphQL API service for creating and managing trips, participants, and expense records.

-----

## 2\. Core Features

  - **Dual-Mode Operation:**
      - **CLI Mode:** Quickly and easily calculates expense-splitting results from a CSV file.
      - **Web Mode:** Offers a full-featured GraphQL API for CRUD (Create, Read, Update, Delete) operations on trips, participants, and expense records.
  - **Real-Time Updates:** The web mode supports GraphQL Subscriptions, allowing clients to receive real-time notifications of changes to trip data.
  - **Core Algorithm:** The code in the `tx` directory is responsible for the core expense-splitting logic, calculating the final transactions (who should pay whom).

-----

## 3\. Technology Stack

  - **Backend Language:** Go (Version 1.23.3)
  - **Web Framework:** Gin
  - **API:** GraphQL (using the `gqlgen` library)
  - **Database:** PostgreSQL
  - **ORM:** GORM
  - **Database Migration:** `pressly/goose`
  - **Command-Line Interface:** `spf13/cobra`
  - **Message Queue (MQ):**
      - RabbitMQ
      - Google Cloud Pub/Sub
      - (This suggests the project may adopt a microservices or event-driven architecture)
  - **Frontend E2E Testing:** Jest (located in the `e2e` directory)
  - **Infrastructure as Code (IaC):** Terraform (located in the `infra` directory)
  - **Containerization:** Docker

-----

## 4\. Project Structure Analysis

```
.
├── cmd/          # Application entry points (server, cli, migrate)
├── config/       # Project configuration files
├── db/           # Database-related logic (pg, mem)
├── e2e/          # End-to-end tests (Jest)
├── graph/        # GraphQL core (schema, resolvers, generated code)
├── infra/        # Infrastructure configuration (Terraform)
├── migration/    # Database migration files
├── mq/           # Message Queue implementations (gcppubsub, rabbit)
├── tx/           # Core expense-splitting algorithm
├── web/          # Web server (Gin handler, middleware)
├── dtm.go        # Main project entry point
├── go.mod        # Go module dependencies
├── Makefile      # Development scripts
├── dockerfile    # Docker container configuration
└── README.md     # Project documentation
```

  - **`cmd/`**: Contains the main entry points for the application. `server.go` starts the GraphQL web service, `share.go` handles the CLI mode, and `migrate.go` is used for running database migrations.
  - **`graph/`**: This is the core of the GraphQL API. `schema.graphqls` defines the API's structure, while `resolver.go` and `schema.resolvers.go` contain the business logic that implements the API.
  - **`db/`**: Responsible for all database interactions. It is divided into different backend implementations like `pg` (PostgreSQL) and `mem` (in-memory), a structure that makes it easy to swap or test databases.
  - **`tx/`**: Contains the project's most critical algorithm, which calculates the most simplified transaction plan based on all expense records.
  - **`mq/`**: Contains integration code for message queue services (like RabbitMQ, GCP Pub/Sub), indicating the system might handle asynchronous tasks via an event-driven approach (e.g., notifying other systems after a record is updated).
  - **`e2e/`**: Contains end-to-end tests written in JavaScript (Jest) to test the functionality of the GraphQL API from a user's perspective.
  - **`infra/`**: Stores Terraform code for automatically provisioning the cloud infrastructure required for the project.
  - **`Makefile`**: Provides a series of convenient development commands, such as `make test` (run unit tests), `make serve` (start the service), `make gql` (regenerate GraphQL code), etc.

-----

## 5\. GraphQL API Design

Based on `graph/schema.graphqls`, the main models of the API are as follows:

  - **`Trip`**: Represents a trip, containing a name, all expense records (`records`), the calculated money flow (`moneyShare`), and a list of participants (`addressList`).
  - **`Record`**: Represents an expense, including a name, amount, the payer (`prePayAddress`), and those who should share the cost (`shouldPayAddress`).
  - **`Tx`**: Represents a single transaction, specifying who (`input`) should pay whom (`output`) and the amount.
  - **`Query`**: Used for querying the details of a trip.
  - **`Mutation`**: Used for performing write operations, such as creating/updating a trip, adding/deleting expense records, and adding/deleting participants.
  - **`Subscription`**: Used for listening to real-time data changes for a specific trip. For example, when a new expense record or participant is added, the client can be notified immediately.

-----

## 6\. Development and Deployment Workflow

  - **Development:**
      - Developers can use `make` commands to perform common tasks, such as formatting code (`make format`), running tests (`make test`), and starting a local development server (`make serve`).
      - The `make dev-docker` command can quickly launch Docker containers for PostgreSQL and RabbitMQ, facilitating local development.
      - GraphQL code is auto-generated via the `make gql` command, ensuring consistency between resolvers and the schema.
  - **Testing:**
      - **Unit Tests:** `make test` executes the Go unit tests.
      - **E2E Tests:** `make testE2E` runs the Jest tests in the `e2e` directory to verify the behavior of the entire system.
  - **Deployment:**
      - The `dockerfile` and the `docker-build` command indicate that the project can be packaged into a Docker image for deployment.
      - The Terraform code in the `infra/` directory implies that the project can be deployed to a cloud platform (likely GCP, given the Pub/Sub integration).
      - The `CI.yml` and `CD.yaml` files under `.github/workflows` show that the project has set up Continuous Integration (CI) and Continuous Deployment (CD) pipelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leon123858)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leon123858)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
