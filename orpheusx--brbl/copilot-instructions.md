## brbl

> This project, internally named "Burble," is a robust, data-driven messaging platform designed for SaaS operations. It handles the lifecycle of messages from reception (MO - Mobile Originated) to delivery (MT - Mobile Terminated) through a decoupled architecture.

# Gemini Context: sndrRcvr (Burble)

This project, internally named "Burble," is a robust, data-driven messaging platform designed for SaaS operations. It handles the lifecycle of messages from reception (MO - Mobile Originated) to delivery (MT - Mobile Terminated) through a decoupled architecture.

## Project Overview

- **Main Technologies:** Java 23 (utilizing preview features like Virtual Threads and Structured Concurrency), Helidon SE (Web Server & Client), RabbitMQ (Messaging Broker), PostgreSQL (Persistence).
- **Architecture:**
  - **Rcvr (Receiver):** Entry point for incoming messages (MO). Enqueues to RabbitMQ.
  - **Opr8r (Operator):** Core logic engine. Dequeues MOs, processes them based on stateful "Scripts" (conversational graphs), and enqueues responses (MTs).
  - **Sndr (Sender):** Dequeues MTs from RabbitMQ and delivers them to external gateways (e.g., SMS, WhatsApp).
  - **BrblCli:** A management CLI for system interaction and simulation.
  - **Chttr:** A client that reads "Scripts" and uses them to simulate end-user interaction with the platform.
- **Messaging Conventions:** RabbitMQ queues follow the pattern `<platform>.<region>.<direction>` (e.g., `whatsapp.us.mo`, `test.local.mt`).

## Building and Running

### Prerequisites
- **Java 23 JDK** (Must support `--enable-preview`).
- **Docker** (For RabbitMQ, Postgres, and containerized deployment).
- **Maven 3.x**.

### Build Commands
- **Full Executable JAR:** `mvn clean compile assembly:single` (produces a `jar-with-dependencies` in `target/`).
- **Native Image Build:** `mvn -Pnative package` (requires GraalVM).
- **Docker Rebuild Helper:** `./rbc.sh` (Rebuilds the JAR and the JVM-based Docker image).

### Running Locally
- **Start Dependencies:** Use the provided scripts like `startPostgres.sh` or run RabbitMQ via Docker:
  ```bash
  docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
  ```
- **Run Application Components:**
  - `java --enable-preview -jar target/sndrRcvr-1.0-SNAPSHOT-jar-with-dependencies.jar Rcvr`
  - `java --enable-preview -jar target/sndrRcvr-1.0-SNAPSHOT-jar-with-dependencies.jar Sndr`
  - `java --enable-preview -jar target/sndrRcvr-1.0-SNAPSHOT-jar-with-dependencies.jar FakeOperator`

## Development Conventions

- **Java Preview Features:** Always use `--enable-preview` in compiler and runtime arguments.
- **Database Schema:**
  - `brbl_logic`: Conversational nodes, edges, and keywords.
  - `brbl_logs`: Message history and processing logs.
  - `brbl_users`: User profiles and session data.
- **Testing Strategy:**
  - **Unit Tests:** JUnit 5.
  - **Integration Tests:** Use Testcontainers for RabbitMQ and Postgres. Look for `*IT.java` files in `src/test/java`.
  - **Simulation:** Use `FakeOperator` and `PlatformGateway` to simulate external environments.
- **Logging:** Uses SLF4J with Logback. Configuration is provided by com.enoughisasgoodasafeast.TylerConfigurator, a class generated from `src/main/resources/logback.xml`. The runtime only uses the class, not the config file. See the commented block of the top-level pom.xml file that includes the dependency, ch.qos.logback.tyler::tyler-base.
- **Coding Standards & Style Guide:** 
  - Follow existing style conventions like camelCase method names. 
  - Use 4 spaces for indentation. 
  - If a method can return null annotate it with the JSpecify @Nullable. 
  - If a method always returns a non-null value annotate with JSpecify @NonNull.
  - If a method parameter is used without checking for null, annotate its declaration with @NonNull. Otherwise, annotate it with @Nullable.
  - Always try to compile any new Java code before declaring them correct or complete.

## Key Directories

- `src/main/java/com/enoughisasgoodasafeast/`: Core source code.
  - `Burble.java`: Entry point that dispatches to specific programs.
  - `datagen/`: Tools for generating test data.
  - `operator/`: Conversational logic and state machine.
- `ddl/migrations/`: Database migration scripts (using pgroll/manual SQL).
- `bin/`: Shell scripts for starting specific components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orpheusx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/orpheusx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
