## my-rpc-cc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Java RPC (Remote Procedure Call) framework called "yu-rpc" built with modern technologies including Vert.x, Etcd, and ZooKeeper. The project demonstrates advanced networking concepts, custom protocols, load balancing, fault tolerance, and Spring Boot integration.

## Build Commands

### Building the Project
```bash
# Build all modules
mvn clean install

# Build specific module
mvn clean install -pl yu-rpc-core

# Skip tests during build
mvn clean install -DskipTests

# Build only yu-rpc-easy module
mvn clean install -pl yu-rpc-easy
```

### Running Examples
```bash
# Start service provider (standalone)
cd example-provider
mvn exec:java -Dexec.mainClass="com.yupi.example.provider.ProviderExample"

# Start service consumer (standalone)
cd example-consumer
mvn exec:java -Dexec.mainClass="com.yupi.example.consumer.ConsumerExample"

# Start Spring Boot provider
cd example-springboot-provider
mvn spring-boot:run

# Start Spring Boot consumer
cd example-springboot-consumer
mvn spring-boot:run

# Run easy examples
cd yu-rpc-easy
mvn exec:java -Dexec.mainClass="com.yupi.yurpc.EasyProviderExample"
mvn exec:java -Dexec.mainClass="com.yupi.yurpc.EasyConsumerExample"
```

### Testing
```bash
# Run all tests
mvn test

# Run tests for specific module
mvn test -pl yu-rpc-core

# Run single test class
mvn test -pl yu-rpc-core -Dtest=ProtocolMessageTest

# Run specific test method
mvn test -pl yu-rpc-core -Dtest=ProtocolMessageTest#testEncode
```

## Architecture

### Core Modules
- **yu-rpc-core**: Main RPC framework implementation with networking, serialization, and service discovery
- **yu-rpc-easy**: Simplified version suitable for beginners
- **yu-rpc-spring-boot-starter**: Spring Boot integration with annotation-driven development

### Example Modules
- **example-common**: Shared service interfaces and models
- **example-provider/consumer**: Standalone service provider and consumer
- **example-springboot-provider/consumer**: Spring Boot integrated examples

### Key Components

#### Service Registry
- Supports both Etcd and ZooKeeper for service discovery
- Heartbeat detection and service health monitoring
- Client-side caching with automatic updates via watch mechanisms

#### Custom Protocol
- TCP-based communication using Vert.x
- Custom message format with header and body
- Handles sticky packets and half-packets
- Multiple serialization formats (JSON, Kryo, Hessian, JDK)

#### Load Balancing
- Round Robin, Random, and Consistent Hash algorithms
- Extensible via SPI mechanism

#### Fault Tolerance
- Multiple retry strategies (No retry, Fixed interval, etc.)
- Fault tolerance strategies (Fail Fast, Fail Over, Fail Safe, etc.)

#### Configuration
- Global configuration management via `RpcApplication`
- Properties-based configuration with fallback defaults
- Registry configuration for service discovery

### SPI Extensions
The framework uses dual SPI configuration system for extensibility:

**Standard Java SPI** (Location: `META-INF/services/`):
- Currently configured: JdkSerializer

**Custom SPI Configuration** (Location: `META-INF/rpc/system/`):
- Serializers: jdk, hessian, json, kryo
- Load Balancers: random, round_robin, consistent_hash
- Retry Strategies: no, fixed_interval
- Tolerant Strategies: fail_fast, fail_over, fail_safe, fail_back
- Registries: etcd, zookeeper

### Development Patterns
- Factory pattern for creating strategy instances
- Decorator pattern for client enhancement
- Double-checked locking singleton for configuration management
- Proxy pattern for service invocation

## Development Notes

### Adding New Components
1. Create interface in appropriate package
2. Implement the interface with @Component annotation
3. Add SPI configuration file under `META-INF/rpc/system/` (preferred) or `META-INF/services/`
4. Implement corresponding factory class if needed
5. Add configuration keys constant

### Spring Boot Integration
- Use `@EnableRpc` annotation to enable RPC framework
- Configure via `application.properties` with `rpc.` prefix
- Supports `needServer` parameter to control server startup
- Automatic bootstrap integration with Spring Boot lifecycle

### Testing Strategy
- Unit tests for individual components (load balancer, retry, protocol, etc.)
- Integration tests using example modules
- Mock service proxy for isolated testing
- Test utilities: `ProtocolMessageTest`, `LoadBalancerTest`, `RetryStrategyTest`

### Configuration Management
- Default configuration loaded from `RpcConstant.DEFAULT_CONFIG_PREFIX`
- Custom configuration via `RpcApplication.init(RpcConfig)`
- Registry configuration separate from RPC framework config
- Example configurations in `example-*/src/main/resources/application.properties`

### Key Versions
- Java: 8 (target)
- Vert.x: 4.5.1
- Spring Boot: 2.6.13
- Hutool: 5.8.16
- Lombok: 1.18.30

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xiii-123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xiii-123)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
