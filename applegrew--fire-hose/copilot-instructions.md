## fire-hose

> FireHose is a **Generic Real-time Alerting Engine** designed for high-velocity event ingestion and dynamic rule evaluation. It enables proactive monitoring by matching a live event firehose against broadcasted user rules without requiring code redeploys.

# FireHose: Project Sentinel

## Project Overview
FireHose is a **Generic Real-time Alerting Engine** designed for high-velocity event ingestion and dynamic rule evaluation. It enables proactive monitoring by matching a live event firehose against broadcasted user rules without requiring code redeploys.

![FireHose System Design](./FireHose.jpg)

## Tech Stack
- **Container Runtime:** Colima (open-source Docker Desktop alternative)
- **Streaming Bus:** Apache Kafka 4.1.1 (Strimzi Operator, KRaft mode)
- **Processing Engine:** Apache Flink (Stateful Stream Processing)
- **Orchestration:** Kubernetes (Minikube with Flink Kubernetes Operator)
- **API Gateway:** Kong (local) / AWS API Gateway (production)
- **Logic:** Broadcast State Pattern for dynamic rule evaluation
- **Event Ingester:** Quarkus 3.x microservice (Gradle, SmallRye Reactive Messaging)
- **Query Service:** Quarkus 3.x microservice (Hibernate ORM Panache, PostgreSQL, Redis caching)

## Quick Start
```bash
# Full infrastructure setup (idempotent, safe to re-run)
./development/setup.sh

# Run infrastructure tests only
./development/setup.sh --test

# Build and deploy application services
./development/start.sh

# Deploy services without rebuilding (use existing images)
./development/start.sh --skip-build

# Force rebuild + redeploy and restart pods (useful when using :latest images)
./development/start.sh --force
```

> **Dev seeding note:** `./development/setup.sh` seeds PostgreSQL with development-only data from `development/import-dev.sql`. Application services do not seed the database on startup.

## Project Structure
```
fire-hose/
├── development/
│   ├── setup.sh              # Infrastructure setup (Colima, Minikube, Kafka, Flink, Kong)
│   ├── start.sh              # Application services deployment (event-ingester, query-svc)
│   ├── import-dev.sql         # Dev-only PostgreSQL seed data (applied by setup.sh)
│   └── k8s/
│       ├── kafka-kraft.yaml  # Kafka cluster configuration
│       ├── kong-config.yaml  # Kong routes and plugins
│       └── kong-values.yaml  # Kong Helm values
├── gateway/
│   ├── gateway-config.yaml   # Provider-agnostic config
│   ├── kong/                 # Local development (Kong)
│   └── aws/                  # Production (AWS API Gateway)
├── event-ingester/           # Event ingestion microservice
├── query-svc/                # Query/CRUD microservice with caching
└── laptop-metrics-simulator/ # Laptop metrics simulator (OpenTelemetry, load testing)
```

## Kafka Topics
| Topic | Purpose |
|-------|---------|
| `events` | Raw ingested events from API Gateway |
| `events_dlq` | Dead letter queue for failed transformations |
| `alerts` | Generated alerts from Flink rule evaluation |
| `notifications` | Notification requests for Email/SMS delivery |

## Core Commands
### Infrastructure Management
- Start Colima: `colima start --cpu 4 --memory 10 --disk 60`
- Stop Colima: `colima stop`
- Start Minikube: `minikube start --cpus=4 --memory=8192`
- Stop Minikube: `minikube stop`

### Development Workflow
- Forward Kafka: `kubectl port-forward svc/my-cluster-kafka-bootstrap 9092:9092 -n kafka`
- Forward Flink UI: `kubectl port-forward svc/firehose-alert-engine-rest 8081:8081 -n flink`
- Forward Event Ingester: `kubectl port-forward svc/event-ingester 8080:8080 -n firehose`
- Forward Query Service: `kubectl port-forward svc/query-svc 8081:8080 -n firehose`
- Forward Kong Proxy: `kubectl port-forward svc/kong-kong-proxy 8000:80 -n kong`
- Forward Kong Admin: `kubectl port-forward svc/kong-kong-admin 8001:8001 -n kong`
- Kong Proxy URL (Minikube): `minikube service kong-kong-proxy -n kong --url`
- List Topics: `kubectl -n kafka run topic-list --image=quay.io/strimzi/kafka:latest-kafka-4.1.1 --rm=true --restart=Never -i -- bin/kafka-topics.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --list`

### Cluster Status
- Check all pods: `kubectl get pods -A`
- Kafka status: `kubectl get kafka -n kafka`
- Flink operator: `kubectl get pods -n flink`
- Kong Gateway: `kubectl get pods -n kong`
- Kong services: `curl localhost:8001/services` (after port-forward)

## Service Architecture

### External Services (Exposed via Kong Gateway)
| Service | Purpose | Kong Route | Auth Required |
|---------|---------|------------|---------------|
| Event Ingester | Generic event ingestion (any JSON format) | `POST /events` | Yes (API Key) |
| Event Ingester | Health checks | `GET /events/health`, `GET /q/health` | No |

### Internal Services (Cluster-only, NOT exposed via Gateway)
| Service | Purpose | Internal Access |
|---------|---------|-----------------|
| Query Service | CRUD operations, API key validation, caching | `query-svc.firehose.svc.cluster.local:8080` |
| PostgreSQL | Persistent storage for rules, keys, alerts | `postgresql.firehose.svc.cluster.local:5432` |
| Redis | Caching for API keys and event types | `redis.firehose.svc.cluster.local:6379` |
| Kafka | Event streaming bus | `my-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092` |
| Flink | Stream processing engine | `firehose-alert-engine-rest.flink.svc.cluster.local:8081` |

> **Note:** Query Service is used internally by Kong's pre-function plugin for API key validation but is NOT accessible externally.

## Event Ingester Service (External)
Quarkus-based microservice that ingests events from the API Gateway and produces them to Kafka. Accepts **any JSON payload** - the event type (determined by API key) tells the system how to interpret the data.

### Location
`event-ingester/`

### Event Ingestion Flow
```
Client → Kong (X-API-Key header)
           ↓
         Kong pre-function validates API key via Query Service
           ↓
         Query Service returns: eventTypeName, eventTypeTableName
           ↓
         Kong sets headers: X-Event-Type, X-Table-Name, X-Validated-API-Key
           ↓
         Event Ingester receives generic JSON + headers
           ↓
         Creates EventEnvelope with metadata → Kafka
```

### Build & Run
```bash
# Development mode (hot reload)
cd event-ingester && ./gradlew quarkusDev

# Build JAR
./gradlew build

# Build container image
./gradlew build -Dquarkus.container-image.build=true

# Deploy to Kubernetes
kubectl apply -f event-ingester/k8s/deployment.yaml
```

### API
```bash
# Ingest an event (any JSON payload - format determined by API key's event type)
curl -X POST http://localhost:8080/events \
  -H "Content-Type: application/json" \
  -H "X-API-Key: my-api-key" \
  -d '{"cpu": 45.5, "memory": 8192, "timestamp": "2024-01-01T00:00:00Z"}'

# Health check
curl http://localhost:8080/q/health
```

### Headers Set by Kong
| Header | Description |
|--------|-------------|
| `X-Event-Type` | Event type name from API key (e.g., "otlp", "user.login") |
| `X-Table-Name` | Target time series table name |
| `X-Validated-API-Key` | The validated API key value |

### Configuration
- Kafka bootstrap: `KAFKA_BOOTSTRAP_SERVERS` env var (default: `localhost:9092`)
- In K8s: connects to `my-cluster-kafka-bootstrap.kafka:9092`

## Query Service (Internal)
Quarkus-based microservice providing REST CRUD operations to the RDBMS with Redis caching for frequently accessed data (api_key, event_type).

> **Internal Service:** Not exposed via Kong Gateway. Access via port-forward for local development or internal cluster DNS for service-to-service communication.

### Location
`query-svc/`

### Database Schema
| Entity | Purpose |
|--------|---------|
| `EventType` | Event type definitions (name, format JSON, target table) |
| `EventTypeMap` | Field mappings for event types |
| `ApiKey` | API keys with optional event type binding and TTL |
| `User` | Users for notification targeting |
| `EventRule` | Rules for matching events and generating alerts |
| `NotificationRule` | Notification configuration per event rule |
| `Alert` | Generated alerts from rule matches |

### Build & Run
```bash
# Development mode (requires PostgreSQL and Redis running)
cd query-svc && ./gradlew quarkusDev

# Build JAR
./gradlew build

# Build container image
./gradlew build -Dquarkus.container-image.build=true

# Deploy infrastructure (PostgreSQL + Redis)
kubectl apply -f query-svc/k8s/infrastructure.yaml

# Deploy service
kubectl apply -f query-svc/k8s/deployment.yaml
```

### API Endpoints
```bash
# Event Types
curl http://localhost:8080/api/event-types
curl http://localhost:8080/api/event-types/lookup/user.login  # Cached lookup

# API Keys
curl http://localhost:8080/api/api-keys
curl http://localhost:8080/api/api-keys/validate/fh_dev_key_001  # Cached validation
curl -X POST http://localhost:8080/api/api-keys \
  -H "Content-Type: application/json" \
  -d '{"ttlSeconds": 3600}'

# Users, Event Rules, Alerts
curl http://localhost:8080/api/users
curl http://localhost:8080/api/event-rules
curl http://localhost:8080/api/alerts/stats

# OpenAPI/Swagger
curl http://localhost:8080/q/openapi
open http://localhost:8080/q/swagger-ui
```

### Configuration
- Database: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- Redis: `REDIS_HOST`, `REDIS_PORT`
- In K8s: PostgreSQL at `postgresql.firehose:5432`, Redis at `redis.firehose:6379`

## API Gateway

### Overview
The API Gateway handles authentication, rate limiting, and routing. Kong is used locally to mirror AWS API Gateway behavior in production.

**API keys are validated against PostgreSQL via Query Service** - no static keys in Kong config.

| Feature | Kong (Local) | AWS API Gateway (Prod) |
|---------|--------------|------------------------|
| API Key Auth | `pre-function` → Query Service | API Keys + Usage Plans |
| Rate Limiting | `rate-limiting` plugin | Usage Plan throttling |
| Request ID | `correlation-id` plugin | `$context.requestId` |
| Max Payload | `request-size-limiting` | 10MB default |

### API Key Validation Flow
```
Request → Kong → pre-function plugin → Query Service /api/api-keys/validate/{key}
                                              ↓
                                         PostgreSQL (ApiKey → EventType)
                                              ↓
                              Valid? → Set headers (X-Event-Type, X-Table-Name) → Forward to backend
                              Invalid? → Return 401
```

The validation response includes:
- `eventTypeName`: The event type associated with this API key
- `eventTypeTableName`: The target time series table for this event type

### Directory Structure
```
gateway/
├── gateway-config.yaml     # Provider-agnostic config (source of truth)
├── kong/
│   └── configure.sh        # Kong configuration script
└── aws/
    └── main.tf             # Terraform for AWS API Gateway
```

### Local Development (Kong)
```bash
# Get Kong proxy URL
KONG_URL=$(minikube service kong-kong-proxy -n kong --url | head -1)

# First, create an event type via Query Service
curl -X POST http://localhost:8081/api/event-types \
  -H "Content-Type: application/json" \
  -d '{"name":"laptop-metrics","tableName":"laptop_metrics"}'

# Create an API key linked to the event type
curl -X POST http://localhost:8081/api/api-keys \
  -H "Content-Type: application/json" \
  -d '{"keyValue":"my-new-key","ttlSeconds":3600,"eventType":{"id":1}}'

# Ingest event via Kong (any JSON format - determined by API key's event type)
curl -X POST "$KONG_URL/events" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: my-new-key" \
  -d '{"cpu": 45.5, "memory": 8192, "hostname": "laptop-001"}'

# Health check (no auth required)
curl "$KONG_URL/events/health"
```

### Managing API Keys (Internal Access)
API keys are managed via the Query Service CRUD API (access via port-forward):
```bash
# Start port-forward to Query Service
kubectl port-forward svc/query-svc 8081:8080 -n firehose
```
```bash
# Create API key
curl -X POST http://localhost:8081/api/api-keys \
  -H "Content-Type: application/json" \
  -d '{"keyValue":"fh_prod_key_001","ttlSeconds":86400}'

# List all API keys
curl http://localhost:8081/api/api-keys

# Validate an API key (used by Kong)
curl http://localhost:8081/api/api-keys/validate/fh_prod_key_001

# Deactivate an API key
curl -X POST http://localhost:8081/api/api-keys/1/deactivate
```

### Production Deployment (AWS)
```bash
cd gateway/aws

# Initialize Terraform
terraform init

# Deploy (requires VPC Link and NLB to be set up)
terraform apply \
  -var="environment=production" \
  -var="vpc_link_id=abc123" \
  -var="nlb_dns_name=internal-firehose-nlb.us-east-1.elb.amazonaws.com"
```

### Swapping Environments
The `gateway/gateway-config.yaml` file is the canonical configuration. When making changes:
1. Update `gateway-config.yaml` first
2. Apply changes to `gateway/kong/configure.sh` for local dev
3. Apply changes to `gateway/aws/main.tf` for production

## Architectural Guidelines
- **Dynamic Rules:** Use the Broadcast State Pattern to propagate rules to all TaskManagers.
- **Auto-scaling:** Configure the Flink Kubernetes Operator to monitor `consumer-lag` and `busyTimeMsPerSecond`.
- **State Management:** Use RocksDB as the state backend for handling high-volume windows locally.

## Scaling Constraints
- Flink parallelism cannot exceed the number of Kafka partitions (currently 6).
- Scaling events should prioritize clearing Kafka lag after a job restart.
- Replication factor is 1 (single broker dev setup); use 3 for production.

## Generic Event Ingestion

The Event Ingester accepts **any JSON payload format**. The API key determines the event type, which tells the system:
- What format to expect (via `formatJson` field)
- Which time series table to target (via `tableName` field)
- Field mappings for extracting key-value pairs (via `EventTypeMap` entries)

### Setting Up a New Event Type
```bash
# Port-forward to Query Service
kubectl port-forward svc/query-svc 8081:8080 -n firehose

# 1. Create the event type
curl -X POST http://localhost:8081/api/event-types \
  -H "Content-Type: application/json" \
  -d '{
    "name": "otlp",
    "formatJson": "{\"resourceMetrics\":[]}",
    "tableName": "otlp_metrics"
  }'

# 2. Create an API key linked to this event type
curl -X POST http://localhost:8081/api/api-keys \
  -H "Content-Type: application/json" \
  -d '{
    "keyValue": "otlp-api-key-001",
    "ttlSeconds": 86400,
    "eventType": {"id": 1}
  }'
```

### EventEnvelope (Kafka Message)
Events are wrapped in an envelope before being sent to Kafka:
```json
{
  "id": "uuid",
  "apiKey": "the-api-key",
  "eventType": "otlp",
  "tableName": "otlp_metrics",
  "receivedAt": "2024-01-01T00:00:00Z",
  "payload": { "<raw JSON payload>" }
}
```

## Laptop Metrics Simulator (Load Testing)
A standalone Java application that generates simulated laptop metrics in OpenTelemetry JSON format for load testing the FireHose system.

### Location
`laptop-metrics-simulator/`

### Purpose
Simulates metrics from multiple laptops (CPU, Memory, GPU, Battery, Disk, Processes) to test the FireHose system's ability to handle high-volume metric ingestion from endpoint monitoring agents.

### Quick Start
```bash
cd laptop-metrics-simulator

# Port-forward Event Ingester (direct, bypassing Kong)
kubectl port-forward svc/event-ingester 8080:8080 -n firehose &

# Run with default config (10 laptops)
./gradlew run

# Load testing with 100 laptops, faster intervals
LAPTOP_COUNT=100 \
INTERVAL_HIGH_FREQ=10 \
INTERVAL_MED_FREQ=30 \
./gradlew run

# With API key (when going through Kong)
API_KEY=otlp-api-key-001 \
OTLP_ENDPOINT=http://localhost:8000/events \
./gradlew run
```

### Configuration
| Variable | Default | Description |
|----------|---------|-------------|
| `LAPTOP_COUNT` | 10 | Number of simulated laptops |
| `OTLP_ENDPOINT` | `http://localhost:8080/events` | Event Ingester endpoint |
| `API_KEY` | (none) | API key for authentication (required via Kong) |
| `INTERVAL_HIGH_FREQ` | 60 | High frequency metrics interval (seconds) |
| `INTERVAL_MED_FREQ` | 300 | Medium frequency metrics interval (seconds) |
| `INTERVAL_LOW_FREQ` | 3600 | Low frequency metrics interval (seconds) |

### Setup for Simulator
Before running the simulator through Kong, create the event type and API key:
```bash
# Create otlp event type
curl -X POST http://localhost:8081/api/event-types \
  -H "Content-Type: application/json" \
  -d '{"name":"otlp","tableName":"otlp_metrics"}'

# Create API key for the simulator
curl -X POST http://localhost:8081/api/api-keys \
  -H "Content-Type: application/json" \
  -d '{"keyValue":"otlp-api-key-001","eventType":{"id":1}}'
```

### Metrics Generated
- **High Frequency (1m):** CPU usage, Memory usage, GPU usage, Process CPU/Memory
- **Medium Frequency (5m):** Battery charge/health/charging, Process count
- **Low Frequency (1hr):** Disk free space, System uptime

See `laptop-metrics-simulator/README.md` for detailed documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/applegrew)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/applegrew)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
