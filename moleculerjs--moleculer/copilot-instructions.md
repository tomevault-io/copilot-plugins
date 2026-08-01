## moleculer

> **Fast & powerful microservices framework for Node.js**

# Moleculer Framework

**Fast & powerful microservices framework for Node.js**

## Project Overview

Moleculer is a modern, production-ready microservices framework for Node.js that enables you to build efficient, reliable, and scalable distributed systems. It provides a comprehensive set of features for microservices architecture including service discovery, load balancing, fault tolerance, and more.

### Key Features
- Promise-based solution with async/await support
- Request-reply concept with event-driven architecture
- Built-in service registry and dynamic service discovery
- Load balanced requests and events (round-robin, random, CPU usage, latency, sharding)
- Comprehensive fault tolerance features (Circuit Breaker, Bulkhead, Retry, Timeout, Fallback)
- Plugin/middleware system
- Versioned services support
- Node.js Streams support
- Service mixins
- Built-in caching solutions (Memory, MemoryLRU, Redis)
- Metrics and tracing capabilities
- Multiple transporters (TCP, NATS, MQTT, AMQP, Redis, Kafka)
- Multiple serializers (JSON, Avro, MessagePack, CBOR)

## Technology Stack

### Core Technologies
- **Node.js**: Version 20.x.x or higher (required)
- **TypeScript**: Full TypeScript support with type definitions
- **JavaScript ES2022**: Modern JavaScript features
- **CommonJS**: Primary module system with ESM support

### Dependencies
- **Core Dependencies**: args, eventemitter2, fastest-validator, glob, ipaddr.js, kleur, lodash, lru-cache, recursive-watch
- **Transport Layer**: Multiple optional peer dependencies for different transporters
- **Logging**: Multiple optional logger integrations
- **Monitoring**: Optional tracing and metrics integrations

## Project Structure

```
moleculer/
├── src/                          # Main source code
│   ├── cachers/                  # Caching implementations
│   │   ├── base.js              # Base cacher class
│   │   ├── memory.js            # Memory cache
│   │   ├── memory-lru.js        # LRU memory cache
│   │   └── redis.js             # Redis cache
│   ├── transporters/            # Transport layer implementations
│   │   ├── base.js              # Base transporter class
│   │   ├── amqp.js              # AMQP transporter
│   │   ├── nats.js              # NATS transporter
│   │   ├── redis.js             # Redis transporter
│   │   ├── mqtt.js              # MQTT transporter
│   │   ├── kafka.js             # Kafka transporter
│   │   └── tcp/                 # TCP transporter implementation
│   ├── serializers/             # Data serialization
│   ├── strategies/              # Load balancing strategies
│   ├── middlewares/             # Middleware implementations
│   ├── tracing/                 # Distributed tracing
│   ├── metrics/                 # Metrics collection
│   ├── registry/                # Service registry
│   ├── validators/              # Validation
│   ├── loggers/                 # Logger implementations
│   ├── service-broker.js        # Core broker class
│   ├── service.js               # Service base class
│   ├── context.js               # Request context
│   ├── transit.js               # Message transport
│   ├── utils.js                 # Utility functions
│   ├── errors.js                # Error handling
│   ├── constants.js             # Framework constants
│   └── middleware.js            # Middleware system
├── types/                       # TypeScript type definitions
├── test/                        # Test suite
│   ├── unit/                    # Unit tests
│   ├── integration/             # Integration tests
│   ├── e2e/                     # End-to-end tests
│   ├── leak-detection/          # Memory leak tests
│   ├── typescript/              # TypeScript tests
│   └── services/                # Test services
├── examples/                    # Example implementations
├── docs/                        # Documentation
├── benchmark/                   # Performance benchmarks
├── bin/                         # CLI tools
│   ├── moleculer-runner.js      # Main CLI runner
│   └── moleculer-runner.mjs     # ESM version
├── index.js                     # Main entry point (CommonJS)
├── index.mjs                    # ESM entry point
├── index.d.ts                   # TypeScript definitions
└── package.json                 # Project configuration
```

## Key Entry Points

### Main Entry Points
- **index.js**: CommonJS main entry point - exports all core classes and utilities
- **index.mjs**: ESM entry point for modern module systems
- **index.d.ts**: TypeScript type definitions

### Core Classes and Modules
- **ServiceBroker**: Main broker class that orchestrates the entire system
- **Service**: Base class for creating microservices
- **Context**: Request context handling
- **Transit**: Message transport layer
- **Registry**: Service registry and discovery
- **Cachers**: Caching implementations
- **Transporters**: Message transport implementations
- **Serializers**: Data serialization
- **Strategies**: Load balancing strategies
- **Middlewares**: Middleware system
- **TracerExporters**: Distributed tracing exporters
- **MetricTypes/MetricReporters**: Metrics collection and reporting

## Development Workflow

### Available Scripts
```bash
# Development
npm run dev                    # Start development server with nodemon

# Testing
npm test                       # Run full test suite with coverage
npm run test:unit             # Run unit tests
npm run test:int              # Run integration tests
npm run test:e2e              # Run end-to-end tests
npm run test:leak             # Run memory leak detection
npm run test:ts               # Run TypeScript tests
npm run test:esm              # Run ESM tests

# Code Quality
npm run lint                  # Run ESLint
npm run lint:fix             # Fix ESLint issues

# Type Checking
npm run tsc                  # TypeScript compilation check
npm run tsd                  # TypeScript declaration tests

# Examples and Demos
npm run demo                 # Run examples
npm run demo:ts              # Run TypeScript examples

# Performance
npm run bench                # Run benchmarks
npm run perf                 # Performance profiling
npm run memleak              # Memory leak testing

# Build and Release
npm run release              # Publish to npm
npm run release:beta         # Publish beta version
```

### Testing Strategy
- **Unit Tests**: Individual component testing using Jest
- **Integration Tests**: Component interaction testing
- **E2E Tests**: Full system testing with real transporters
- **Memory Leak Detection**: Specialized tests for memory management
- **TypeScript Tests**: Type safety validation
- **ESM Tests**: ESM module system validation

## Configuration Files

### Build and Development
- **package.json**: Project dependencies, scripts, and metadata
- **tsconfig.json**: TypeScript configuration (ES2022 target, strict mode disabled)
- **jest.config**: Test configuration (in package.json)
- **eslint.config.js**: Code linting rules
- **prettier.config.js**: Code formatting rules
- **.editorconfig**: Editor configuration

### CI/CD
- **.github/workflows/ci.yml**: GitHub Actions CI pipeline
- **Node.js versions**: 20.x, 22.x, 24.x
- **Testing**: Full test suite with coverage reporting
- **Code Quality**: ESLint and security checks

## Code Style and Standards

### JavaScript/TypeScript Standards
- **Target**: ES2022 with ES2023 libraries
- **Module System**: CommonJS primary with ESM support
- **TypeScript**: Strict mode disabled, declaration generation enabled
- **Linting**: ESLint with Node.js, security, and prettier plugins
- **Formatting**: Prettier for consistent code formatting

### Naming Conventions
- **Classes**: PascalCase (ServiceBroker, ServiceFactory)
- **Methods**: camelCase (callService, emitEvent)
- **Constants**: UPPER_SNAKE_CASE (CIRCUIT_CLOSE, MOLECULER_VERSION)
- **Files**: kebab-case or snake-case for utilities, PascalCase for classes

## Key Dependencies

### Core Runtime Dependencies
- **eventemitter2**: Enhanced event emitter
- **lodash**: Utility library
- **fastest-validator**: Data validation
- **glob**: File pattern matching
- **lru-cache**: LRU caching implementation

### Optional Peer Dependencies
- **Transporters**: NATS, MQTT, AMQP, Redis, Kafka
- **Logging**: Winston, Pino, Bunyan, Log4js
- **Tracing**: Jaeger, DataDog, OpenTelemetry
- **Serialization**: Avro, MessagePack, CBOR

## Contributing Guidelines

### Development Setup
1. Clone the repository
2. Install dependencies: `npm install`
3. Run tests: `npm test`
4. Start development: `npm run dev`

### Code Quality Requirements
- All tests must pass
- ESLint must pass without errors
- TypeScript types must be valid
- Memory leak tests must pass
- Code coverage should be maintained

### Version Management
- Follow semantic versioning
- Use conventional commit messages
- Update changelog for releases
- Tag releases with git tags

## Documentation

### Official Documentation
- **Website**: https://moleculer.services
- **API Docs**: https://moleculer.services/docs
- **Examples**: Located in `examples/` directory
- **Migration Guides**: Available in `docs/` directory

### Key Documentation Files
- **README.md**: Project overview and quick start
- **CHANGELOG.md**: Version history and changes
- **CONTRIBUTING.md**: Contribution guidelines
- **CODE_OF_CONDUCT.md**: Community guidelines

## Performance and Scalability

### Performance Characteristics
- Fast message processing with minimal overhead
- Efficient service discovery and load balancing
- Memory-efficient with proper cleanup
- Concurrent request handling
- Scalable to thousands of services

### Optimization Features
- Connection pooling for transporters
- Request batching and streaming
- Caching at multiple levels
- Lazy loading of services
- Hot reload capabilities

## Security Considerations

### Built-in Security Features
- Input validation with fastest-validator
- Secure serialization options
- Authentication and authorization hooks
- Rate limiting and circuit breakers
- Request sanitization

### Development Security
- ESLint security plugin integration
- Dependency vulnerability scanning
- Secure coding practices enforced
- Memory leak detection and prevention

---
> Source: [moleculerjs/moleculer](https://github.com/moleculerjs/moleculer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
