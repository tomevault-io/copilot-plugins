## rpc

> This directory contains an incremental learning project focused on building a Remote Procedure Call (RPC) framework in Java. The project is structured into multiple versions (`version1` through `version6`), each adding new features and complexity to the RPC implementation. It's designed as a teaching aid for understanding RPC concepts and building distributed systems.

# Project Context for RPC-Java

This directory contains an incremental learning project focused on building a Remote Procedure Call (RPC) framework in Java. The project is structured into multiple versions (`version1` through `version6`), each adding new features and complexity to the RPC implementation. It's designed as a teaching aid for understanding RPC concepts and building distributed systems.

## Project Overview

The core goal is to create a Java-based RPC framework from scratch, demonstrating key concepts like service discovery, serialization, network communication, and advanced features like load balancing and fault tolerance. The evolution is documented through six distinct versions:

*   **Version 1**: Basic RPC call, dynamic proxy, Netty for transport, Zookeeper for service registry.
*   **Version 2**: Custom Netty encoders/decoders, local service cache with dynamic updates.
*   **Version 3**: Client-side load balancing, client-side fault tolerance (retry), service whitelisting.
*   **Version 4**: Service rate limiting/degradation, circuit breaker implementation.
*   **Version 5**: SPI (Service Provider Interface) mechanism, configuration, multiple serialization options (Kryo, Hessian, Protostuff), optimized shutdown.
*   **Version 6**: Distributed log tracing, heartbeat detection, retry based on method whitelist.

## Key Technologies

*   **Language**: Java (JDK 17 for versions 5 and 6, JDK 8 for version 1)
*   **Build Tool**: Apache Maven (Multi-module project structure in version 5)
*   **Networking**: Netty (for asynchronous network communication)
*   **Service Discovery**: Apache Zookeeper (via Curator Recipes)
*   **Serialization**: Multiple options including Kryo, Hessian, Protostuff, FastJSON.
*   **Frameworks/Libraries**: Spring Boot (version 5+), Lombok, Guava Retrying, Hutool.
*   **Testing**: JUnit 5

## Building and Running

The project requires Java, Maven, and a running Zookeeper instance. Specific instructions for different versions are in the main `README.md`.

### Prerequisites

1.  **Java**: JDK 17 or higher (Check with `java -version`).
2.  **Maven**: Version 3.6 or higher (Check with `mvn -v`).
3.  **Zookeeper**: Running instance (typically started with `bin/zkServer.sh start` or `bin\\zkServer.cmd` on Windows).

### Building (Version 5 Example)

Navigate to the `version5` directory and run:

```bash
cd /path/to/rpc/version5 # Replace with your actual path
mvn clean install -DskipTests
```

This will compile all modules (`krpc-api`, `krpc-common`, `krpc-core`, `krpc-provider`, `krpc-consumer`) and package them into JAR files located in their respective `target` directories.

### Running (Version 5 Example)

Ensure Zookeeper is running. Then, in separate terminals:

1.  **Start the Provider (Service)**:
    ```bash
    cd /path/to/rpc/version5/krpc-provider
    java -cp "target/krpc-provider-1.0-SNAPSHOT.jar:target/lib/*" com.kama.provider.ProviderTest
    ```
2.  **Start the Consumer (Client)**:
    ```bash
    cd /path/to/rpc/version5/krpc-consumer
    java -cp "target/krpc-consumer-1.0-SNAPSHOT.jar:target/lib/*" com.kama.consumer.ConsumerTest
    ```

*Note: Adjust paths and commands for Windows environments as per the main README.*

## Development Conventions

This is a learning project, so conventions are primarily based on standard Java and Maven practices.

*   **Modular Structure**: Version 5+ uses a Maven multi-module structure:
    *   `krpc-api`: Defines service interfaces.
    *   `krpc-common`: Shared utilities, constants, exceptions.
    *   `krpc-core`: Core RPC logic (serialization, transport, proxy).
    *   `krpc-provider`: Service implementation and provider-side logic.
    *   `krpc-consumer`: Client-side logic, service discovery, proxy creation.
*   **Dependency Management**: Managed via the parent `pom.xml` in `version5`.
*   **Testing**: JUnit 5 is used for unit and integration tests.
*   **Code Style**: Lombok is used extensively to reduce boilerplate code.
*   **Incremental Learning**: Each `versionX` directory represents a snapshot of the project at a specific stage of development, showcasing the addition of new features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kazelise)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kazelise)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
