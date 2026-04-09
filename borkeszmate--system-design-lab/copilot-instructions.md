## system-design-lab

> This repository documents a comprehensive learning path from monolithic applications to distributed system design. The goal is to understand the principles, patterns, and trade-offs involved in building scalable, resilient systems.

# System Design Learning Journey

## Overview
This repository documents a comprehensive learning path from monolithic applications to distributed system design. The goal is to understand the principles, patterns, and trade-offs involved in building scalable, resilient systems.

## Current State
**Background:** Experienced with complex monolithic applications
**Goal:** Master distributed system design and modern architecture patterns
**Approach:** Theory + Practice with hands-on implementations

---

## 📚 Learning Materials (Important for Agents!)

This project contains **extensive educational reference documents** in multiple locations. These files are **comprehensive learning materials** (~625 KB total) that should **NOT be read automatically** to conserve tokens.

### Documentation Locations:

1. **`docs/fundamentals/`** - Phase 1 foundations (119 KB, 4 hours reading)
2. **`docs/phase2-architecture-patterns/`** - Phase 2 patterns (121 KB, 4.5 hours reading)
3. **`docs/phase3-scalability-performance/`** - Phase 3 scalability (50 KB, 2 hours reading)
4. **`docs/phase4-data-management/`** - Phase 4 data management (40 KB, 2 hours reading)
5. **`docs/phase5-reliability-resilience/`** - Phase 5 reliability (50 KB, 2 hours reading)
6. **`docs/phase6-observability-monitoring/`** - Phase 6 observability (50 KB, 2 hours reading)
7. **`docs/phase7-security-identity/`** - Phase 7 security (45 KB, 2 hours reading)
8. **`docs/phase8-deployment-devops/`** - Phase 8 deployment (40 KB, 1.5 hours reading)
9. **`docs/phase9-real-world-systems/`** - Phase 9 case studies (50 KB, 2 hours reading)
10. **`docs/phase10-advanced-topics/`** - Phase 10 advanced (40 KB, 1.5 hours reading)
11. **`ecommerce-microservices/docs/`** - Practical implementation guides

### Instructions for Claude Agents:

**DO NOT:**
- Automatically scan or read files in `docs/fundamentals/`
- Automatically scan or read files in `docs/phase2-architecture-patterns/`
- Automatically scan or read files in `docs/phase3-scalability-performance/`
- Automatically scan or read files in `docs/phase4-data-management/`
- Automatically scan or read files in `docs/phase5-reliability-resilience/`
- Automatically scan or read files in `docs/phase6-observability-monitoring/`
- Automatically scan or read files in `docs/phase7-security-identity/`
- Automatically scan or read files in `docs/phase8-deployment-devops/`
- Automatically scan or read files in `docs/phase9-real-world-systems/`
- Automatically scan or read files in `docs/phase10-advanced-topics/`
- Automatically scan or read files in `ecommerce-microservices/docs/`
- Load these files into context unless explicitly needed
- Include them in exploratory file reads

**ONLY read these files when:**
1. User explicitly asks about a topic covered in the docs
2. User directly references a specific doc file
3. The current task is directly related to the concepts explained
4. User asks to explain a specific pattern or concept

**How to decide if relevant:**
1. First, check the relevant INDEX file:
   - `docs/fundamentals/INDEX.md` - For Phase 1 topics
   - `docs/phase2-architecture-patterns/INDEX.md` - For Phase 2 topics
   - `docs/phase3-scalability-performance/INDEX.md` - For Phase 3 topics
   - `docs/phase4-data-management/INDEX.md` - For Phase 4 topics
   - `docs/phase5-reliability-resilience/INDEX.md` - For Phase 5 topics
   - `docs/phase6-observability-monitoring/INDEX.md` - For Phase 6 topics
   - `docs/phase7-security-identity/INDEX.md` - For Phase 7 topics
   - `docs/phase8-deployment-devops/INDEX.md` - For Phase 8 topics
   - `docs/phase9-real-world-systems/INDEX.md` - For Phase 9 topics
   - `docs/phase10-advanced-topics/INDEX.md` - For Phase 10 topics
   - `ecommerce-microservices/docs/INDEX.md` - For practical guides
2. Read the brief description in the INDEX
3. Only open the full document if it directly answers the user's question
4. Read only the specific section needed, not the entire document

### Available Topics:

#### Phase 1: Fundamentals (docs/fundamentals/)
**Total:** 119 KB across 3 documents + INDEX

- **Monolith Limitations** (~26 KB)
  - Scalability bottlenecks, deployment risks, technology lock-in, team coordination

- **Distributed Systems Fundamentals** (~50 KB)
  - CAP Theorem, ACID vs BASE, Network Fallacies, Consistency Models

- **System Design Principles** (~43 KB)
  - Separation of Concerns, Single Responsibility, Loose Coupling/High Cohesion, Design for Failure

#### Phase 2: Architecture Patterns (docs/phase2-architecture-patterns/)
**Total:** 121 KB across 3 documents + INDEX

- **Microservices Architecture** (~40 KB)
  - Service boundaries, communication patterns, service mesh, API Gateway

- **Event-Driven Architecture** (~43 KB)
  - Event sourcing, CQRS, event streaming vs messaging, Saga pattern

- **Architectural Patterns** (~38 KB)
  - Layered, Hexagonal, Clean Architecture, Strangler Fig pattern

#### Phase 3: Scalability & Performance (docs/phase3-scalability-performance/)
**Total:** 50 KB (1 comprehensive document + INDEX)

- **Scalability & Performance** (~50 KB)
  - Caching strategies (cache-aside, write-through, CDN, Redis patterns)
  - Load balancing (L4 vs L7, algorithms, health checks, circuit breakers)
  - Database scaling (read replicas, sharding, connection pooling)
  - Performance optimization (async processing, denormalization)

#### Phase 4: Data Management (docs/phase4-data-management/)
**Total:** 40 KB (1 comprehensive document + INDEX)

- **Data Management** (~40 KB)
  - Database types (Relational, Document, Key-Value, Column-Family, Graph, Time-Series)
  - Polyglot persistence strategy
  - Data consistency patterns (Two-Phase Commit, Saga, Idempotency)
  - Data replication (Master-slave, Multi-master, Quorum-based)

#### Phase 5: Reliability & Resilience (docs/phase5-reliability-resilience/)
**Total:** 50 KB (1 comprehensive document + INDEX)

- **Reliability & Resilience** (~50 KB)
  - Fault tolerance patterns (Circuit breaker, Bulkhead, Retry, Timeout, Fallback)
  - High availability (Redundancy, health checks, SLAs/SLOs/SLIs, error budgets)
  - Chaos engineering (Principles, experiments, Chaos Monkey, Gremlin, Chaos Mesh)

#### Phase 6: Observability & Monitoring (docs/phase6-observability-monitoring/)
**Total:** 50 KB (1 comprehensive document + INDEX)

- **Observability & Monitoring** (~50 KB)
  - Three pillars (Metrics, Logs, Traces)
  - Monitoring methods (RED, USE, Golden Signals)
  - Tools (Prometheus, Grafana, ELK, OpenTelemetry, Jaeger)
  - Alerting (Design principles, on-call practices, runbooks)

#### Phase 7: Security & Identity (docs/phase7-security-identity/)
**Total:** 45 KB (1 comprehensive document + INDEX)

- **Security & Identity** (~45 KB)
  - Authentication (JWT, sessions, OAuth 2.0, mTLS)
  - Authorization (RBAC)
  - Secrets management (HashiCorp Vault, key rotation)
  - API security (Rate limiting, input validation, CORS, SQL injection prevention)
  - Zero Trust Architecture

#### Phase 8: Deployment & DevOps (docs/phase8-deployment-devops/)
**Total:** 40 KB (1 comprehensive document + INDEX)

- **Deployment & DevOps** (~40 KB)
  - Containerization (Docker, multi-stage builds, Docker Compose)
  - Kubernetes (Deployments, Services, Ingress, HPA, ConfigMaps, Secrets)
  - CI/CD (GitHub Actions, GitLab CI)
  - Deployment strategies (Rolling update, blue-green, canary)
  - Infrastructure as Code (Terraform, Helm)

#### Phase 9: Real-World Systems (docs/phase9-real-world-systems/)
**Total:** 50 KB (1 comprehensive document + INDEX)

- **Real-World Systems** (~50 KB)
  - URL Shortener (Base62 encoding, caching, analytics)
  - Social Media Feed (Hybrid fan-out strategy)
  - Video Streaming (Adaptive bitrate, CDN)
  - E-commerce (Inventory management, Saga, Elasticsearch)
  - Ride-Sharing (Geospatial indexing, matching)
  - Messaging (Delivery guarantees, Cassandra, presence)

#### Phase 10: Advanced Topics (docs/phase10-advanced-topics/)
**Total:** 40 KB (1 comprehensive document + INDEX)

- **Advanced Topics** (~40 KB)
  - Serverless (AWS Lambda, cold start mitigation)
  - Edge Computing (Cloudflare Workers, Lambda@Edge)
  - Modern APIs (GraphQL DataLoader, gRPC streaming)
  - Service Mesh (Istio traffic management, mTLS)
  - Performance (Query optimization, connection pooling, caching)
  - Emerging (WebAssembly, CRDTs, Event Sourcing)

#### Practical Guides (ecommerce-microservices/docs/)
**Total:** ~50 KB across multiple guides

- Docker volumes and data persistence
- Daemons vs processes
- Microservices communication patterns
- Message broker fundamentals (RabbitMQ)
- Queue system comparisons (RabbitMQ vs Celery vs Solace)
- Payment gateway integration patterns

### Examples of Good vs Bad Usage:

**✅ GOOD - Read when needed:**
```
User: "Explain the CAP theorem"
→ Read docs/fundamentals/distributed-systems-fundamentals.md (section 1 only)

User: "How do I implement event sourcing?"
→ Read docs/phase2-architecture-patterns/event-driven-architecture.md (section 1)

User: "What's the Saga pattern?"
→ Read docs/phase2-architecture-patterns/event-driven-architecture.md (section 4)

User: "How do I implement caching?"
→ Read docs/phase3-scalability-performance/scalability-performance.md (section 1)

User: "When should I use MongoDB vs PostgreSQL?"
→ Read docs/phase4-data-management/data-management.md (section 1)

User: "Explain the Saga pattern for distributed transactions"
→ Read docs/phase4-data-management/data-management.md (section 2)
```

**❌ BAD - Don't read automatically:**
```
User: "How do I run the tests?"
→ Don't read any docs, this is a practical question

User: "Debug this error in the code"
→ Don't read theory docs, focus on debugging

User: "List all the files"
→ Don't include docs in file listing, they're reference materials
```

**Why?** These files are very large (~625 KB total) and contain educational content. They should only be loaded when genuinely needed for the current task to conserve tokens and improve response time.

---

## Learning Curve

### Phase 1: Foundation (Weeks 1-2)
**Goal:** Understand why we move beyond monoliths and core distributed system concepts

#### Topics to Cover:
- **Monolith Limitations**
  - Scalability bottlenecks
  - Deployment risks
  - Technology lock-in
  - Team coordination challenges

- **Distributed Systems Fundamentals**
  - CAP Theorem (Consistency, Availability, Partition Tolerance)
  - ACID vs BASE
  - Network fallacies
  - Consistency models (eventual, strong, causal)

- **System Design Principles**
  - Separation of concerns
  - Single Responsibility Principle at system level
  - Loose coupling, high cohesion
  - Design for failure

#### Practical Exercise:
- Document your current monolith's pain points
- Identify bounded contexts using Domain-Driven Design
- Create a service decomposition proposal

---

### Phase 2: Architecture Patterns (Weeks 3-4)
**Goal:** Learn common architectural patterns and when to use them

#### Topics to Cover:
- **Microservices Architecture**
  - Service boundaries and sizing
  - Communication patterns (sync vs async)
  - Service mesh concepts
  - API Gateway pattern

- **Event-Driven Architecture**
  - Event sourcing
  - CQRS (Command Query Responsibility Segregation)
  - Event streaming vs messaging
  - Saga pattern for distributed transactions

- **Other Patterns**
  - Layered architecture
  - Hexagonal architecture (Ports & Adapters)
  - Clean architecture
  - Strangler Fig pattern (for migration)

#### Practical Exercise:
- Design a microservices architecture for your monolith
- Implement a simple event-driven system (2-3 services)
- Build an API Gateway proof-of-concept

---

### Phase 3: Scalability & Performance (Weeks 5-6)
**Goal:** Learn how to scale systems horizontally and vertically

#### Topics to Cover:
- **Caching Strategies**
  - Cache-aside, write-through, write-behind
  - CDN (Content Delivery Networks)
  - Redis, Memcached patterns
  - Cache invalidation strategies

- **Load Balancing**
  - Layer 4 vs Layer 7 load balancing
  - Round-robin, least connections, consistent hashing
  - Health checks and circuit breakers
  - Global and regional load balancing

- **Database Scaling**
  - Read replicas
  - Sharding strategies (horizontal partitioning)
  - Database per service pattern
  - Denormalization for read performance
  - Connection pooling

- **Asynchronous Processing**
  - Message queues (RabbitMQ, Kafka, SQS)
  - Job queues and workers
  - Backpressure handling
  - Dead letter queues

#### Practical Exercise:
- Implement caching layer (Redis) for high-traffic endpoints
- Set up load balancer with multiple instances
- Create a job queue system with workers

---

### Phase 4: Data Management (Weeks 7-8)
**Goal:** Master data storage, consistency, and replication in distributed systems

#### Topics to Cover:
- **Database Types & Use Cases**
  - Relational (PostgreSQL, MySQL)
  - Document stores (MongoDB, CouchDB)
  - Key-value stores (Redis, DynamoDB)
  - Column-family (Cassandra, HBase)
  - Graph databases (Neo4j)
  - Time-series databases (InfluxDB, TimescaleDB)

- **Data Consistency Patterns**
  - Two-phase commit (2PC)
  - Three-phase commit
  - Saga pattern
  - Compensating transactions
  - Idempotency

- **Data Replication**
  - Master-slave replication
  - Multi-master replication
  - Quorum-based replication
  - Conflict resolution

#### Practical Exercise:
- Design a polyglot persistence solution
- Implement a distributed transaction with Saga pattern
- Set up database replication (master-slave)

---

### Phase 5: Reliability & Resilience (Weeks 9-10)
**Goal:** Build systems that gracefully handle failures

#### Topics to Cover:
- **Fault Tolerance Patterns**
  - Circuit breaker (Hystrix, Resilience4j)
  - Bulkhead pattern
  - Retry with exponential backoff
  - Timeout management
  - Fallback strategies

- **High Availability**
  - Redundancy and failover
  - Active-active vs active-passive
  - Health checks and monitoring
  - Disaster recovery planning
  - SLAs, SLOs, SLIs

- **Chaos Engineering**
  - Failure injection
  - Chaos Monkey concepts
  - Testing resilience

#### Practical Exercise:
- Implement circuit breaker pattern
- Set up health checks and auto-recovery
- Conduct a chaos engineering experiment

---

### Phase 6: Observability & Monitoring (Weeks 11-12)
**Goal:** Learn to understand and debug distributed systems in production

#### Topics to Cover:
- **The Three Pillars**
  - Logging (structured logging, log aggregation)
  - Metrics (RED/USE methods, golden signals)
  - Tracing (distributed tracing, OpenTelemetry)

- **Tools & Platforms**
  - ELK Stack (Elasticsearch, Logstash, Kibana)
  - Prometheus + Grafana
  - Jaeger, Zipkin for tracing
  - DataDog, New Relic (commercial options)

- **Alerting & On-Call**
  - Alert design (actionable, not noisy)
  - On-call best practices
  - Incident response

#### Practical Exercise:
- Set up complete observability stack
- Implement distributed tracing across services
- Create meaningful dashboards and alerts

---

### Phase 7: Security & Identity (Weeks 13-14)
**Goal:** Secure distributed systems and manage authentication/authorization

#### Topics to Cover:
- **Authentication & Authorization**
  - OAuth 2.0 / OpenID Connect
  - JWT tokens
  - API keys and secrets management
  - Service-to-service authentication (mTLS)

- **Security Patterns**
  - Zero trust architecture
  - Defense in depth
  - Rate limiting and throttling
  - DDoS protection
  - API security best practices

- **Secrets Management**
  - Vault, AWS Secrets Manager
  - Key rotation
  - Encryption at rest and in transit

#### Practical Exercise:
- Implement OAuth 2.0 authentication
- Set up secrets management solution
- Add rate limiting to API gateway

---

### Phase 8: Deployment & DevOps (Weeks 15-16)
**Goal:** Master deployment strategies and infrastructure as code

#### Topics to Cover:
- **Containerization & Orchestration**
  - Docker deep dive
  - Kubernetes fundamentals
  - Service discovery
  - Container networking

- **CI/CD Pipelines**
  - Automated testing strategies
  - Blue-green deployments
  - Canary releases
  - Rolling deployments
  - Feature flags

- **Infrastructure as Code**
  - Terraform
  - CloudFormation
  - Pulumi
  - GitOps principles

#### Practical Exercise:
- Containerize your services
- Deploy to Kubernetes cluster
- Build complete CI/CD pipeline

---

### Phase 9: Real-World Systems (Weeks 17-18)
**Goal:** Study and understand large-scale production systems

#### Systems to Analyze:
- **URL Shortener** (like bit.ly)
  - Requirements: low latency, high availability
  - Key concepts: hashing, base62 encoding, caching

- **Social Media Feed** (like Twitter)
  - Requirements: real-time updates, high read/write ratio
  - Key concepts: fan-out on write vs read, timelines

- **Video Streaming** (like YouTube)
  - Requirements: bandwidth optimization, global distribution
  - Key concepts: CDN, adaptive bitrate, chunking

- **E-commerce Platform** (like Amazon)
  - Requirements: transactions, inventory, search
  - Key concepts: eventual consistency, order processing

- **Messaging System** (like WhatsApp)
  - Requirements: real-time, delivery guarantees
  - Key concepts: WebSockets, message queues, presence

- **Ride-Sharing** (like Uber)
  - Requirements: geolocation, matching, real-time tracking
  - Key concepts: geohashing, websockets, event streaming

#### Practical Exercise:
- Design 2-3 of these systems in detail
- Implement a simplified version of one system

---

### Phase 10: Advanced Topics (Weeks 19-20)
**Goal:** Explore cutting-edge concepts and optimization techniques

#### Topics to Cover:
- **Advanced Patterns**
  - Serverless architecture (Lambda, Cloud Functions)
  - Edge computing
  - GraphQL federation
  - gRPC and protocol buffers
  - Service mesh (Istio, Linkerd)

- **Performance Optimization**
  - Database query optimization
  - N+1 query problem solutions
  - Connection pooling strategies
  - Batch processing optimization
  - Memory and CPU profiling

- **Global Distribution**
  - Multi-region deployments
  - Data locality and GDPR compliance
  - Global load balancing
  - Conflict-free replicated data types (CRDTs)

#### Practical Exercise:
- Implement serverless functions
- Set up multi-region deployment
- Optimize a bottleneck in your system

---

## Projects Roadmap

### Mini Projects (Throughout)
1. **URL Shortener** - Week 3-4
2. **Task Queue System** - Week 5-6
3. **Chat Application** - Week 7-8
4. **Rate Limiter** - Week 9-10
5. **Notification Service** - Week 11-12

### Capstone Project (Weeks 17-20)
Design and implement a complete distributed system of your choice:
- **Option A:** E-commerce platform with microservices
- **Option B:** Real-time analytics dashboard
- **Option C:** Content delivery and streaming service
- **Option D:** Distributed task scheduler

Requirements:
- 5+ microservices
- Multiple databases (polyglot persistence)
- Message queue integration
- Complete observability
- CI/CD pipeline
- Load testing and optimization
- Documentation and diagrams

---

## Resources

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu (Vol 1 & 2)
- "Building Microservices" by Sam Newman
- "Release It!" by Michael Nygard
- "Database Internals" by Alex Petrov

### Online Resources
- System Design Primer (GitHub)
- High Scalability blog
- AWS Architecture Center
- Google Cloud Architecture Framework
- Microsoft Azure Architecture Center

### Practice Platforms
- LeetCode System Design
- System Design Interview prep sites
- AWS Free Tier for hands-on practice

---

## Progress Tracking

### Completed
- [ ] Phase 1: Foundation
- [ ] Phase 2: Architecture Patterns
- [ ] Phase 3: Scalability & Performance
- [ ] Phase 4: Data Management
- [ ] Phase 5: Reliability & Resilience
- [ ] Phase 6: Observability & Monitoring
- [ ] Phase 7: Security & Identity
- [ ] Phase 8: Deployment & DevOps
- [ ] Phase 9: Real-World Systems
- [ ] Phase 10: Advanced Topics

### Mini Projects
- [ ] URL Shortener
- [ ] Task Queue System
- [ ] Chat Application
- [ ] Rate Limiter
- [ ] Notification Service

### Capstone Project
- [ ] Design document
- [ ] Implementation
- [ ] Testing & optimization
- [ ] Documentation

---

## Notes & Learnings
(Document key insights, challenges, and solutions as you progress)

---

## Next Steps
1. Start with Phase 1: Review monolith limitations and identify decomposition boundaries
2. Set up your learning environment (Docker, local Kubernetes, cloud account)
3. Begin documenting your current system's architecture

**Remember:** System design is about trade-offs. There's no perfect solution, only solutions that fit specific requirements and constraints. Focus on understanding the "why" behind each pattern and practice making reasoned decisions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/borkeszmate)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/borkeszmate)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
