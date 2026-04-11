## camel-sftp-pooled

> This is a Java Maven project implementing a custom Apache Camel component (`pooled-sftp`). It extends the standard Camel SFTP component to support **connection pooling** using `commons-pool2`. This addresses performance bottlenecks in high-throughput scenarios by reusing JSch Sessions and Channels instead of creating a new connection for every file operation.

# Project Context: Custom SFTP Pooled Component

## Overview
This is a Java Maven project implementing a custom Apache Camel component (`pooled-sftp`). It extends the standard Camel SFTP component to support **connection pooling** using `commons-pool2`. This addresses performance bottlenecks in high-throughput scenarios by reusing JSch Sessions and Channels instead of creating a new connection for every file operation.

## Architecture
The solution wraps the standard Camel SFTP logic:
1.  **Component (`PooledSftpComponent`)**: Registers the component and handles configuration.
2.  **Endpoint (`PooledSftpEndpoint`)**: Configures the use of `PooledSftpOperations`.
3.  **Operations (`PooledSftpOperations`)**: The core logic. It borrows a connection from the pool, performs the operation, and returns it.
4.  **Injection (`InjectableSftpOperations`)**: A specialized class that uses reflection to inject the pooled JSch Session/Channel into the standard Camel `SftpOperations`, allowing reuse of the underlying file transfer logic.

## Key Files
*   `src/main/java/com/mycompany/sftp/component/PooledSftpComponent.java`: Main component class. Handles manual property binding.
*   `src/main/java/com/mycompany/sftp/component/PooledSftpOperations.java`: Orchestrates the pool borrowing/returning and delegates to the injectable operations.
*   `src/main/java/com/mycompany/sftp/pool/SftpConnectionFactory.java`: Lifecycle manager for JSch connections (create, validate, destroy).
*   `src/main/java/com/mycompany/app/MainApp.java`: A sample `main` class demonstrating how to bind and use the component in a Camel route.
*   `pom.xml`: Project dependencies (Camel 4.8.0, Commons Pool 2.12.0) and build configuration.

## Building and Running

### Build
To compile and install the project locally:
```bash
mvn clean install
```

### Run Demo
The project includes a `MainApp` that moves files from `target/inbox` to a configured SFTP server.
**Note:** You need a running SFTP server (e.g., localhost:2222) for this to work successfully.

```bash
mvn exec:java -Dexec.mainClass="com.mycompany.app.MainApp"
```
*You may need to modify `MainApp.java` or pass arguments to point to your actual SFTP server.*

## Development Conventions
*   **Java Version**: Java 17+
*   **Camel Version**: 4.x (specifically 4.8.0)
*   **Style**: Standard Java conventions.
*   **Testing**: JUnit 5 and `camel-test-junit5`.
*   **Configuration**: Unlike standard Camel components, this project relies on **manual property binding** in `PooledSftpComponent` rather than generated configurer classes.
*   **Reflection**: The integration relies on accessing private fields (`session`, `channel`) in `SftpOperations`. Future Camel upgrades should be checked for breaking changes in these field names.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seshuthota)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seshuthota)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
