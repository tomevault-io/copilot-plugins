## mt-olympus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mt-Olympus is a high-performance exchange platform with three main microservices written in Go:

- **Apollo** (port 8080): REST API service handling accounts, users, orders, and products
- **Hermes** (port 8000): WebSocket server for real-time messaging and client communication
- **Matcher**: Matching engine that processes orders through an orderbook system

## Architecture

The system uses a microservices architecture with:
- **Message Queue**: Kafka for inter-service communication
- **Database**: PostgreSQL for persistent storage
- **Cache**: Redis for caching (used by Apollo)
- **Protocol Buffers**: Shared data structures for orders and trades
- **WebSockets**: Real-time communication via Hermes

Key data flows:
1. Orders submitted via Apollo REST API
2. Orders published to Kafka and processed by Matcher
3. Trade events published back via Kafka
4. Hermes consumes trade events and broadcasts to WebSocket clients

## Development Commands

### Initial Setup
```bash
# Clone and build protobuf definitions
./build.sh

# Start all services with dependencies
docker-compose up --build
```

### Protocol Buffer Generation
```bash
# Generate Go code from .proto files
protoc --proto_path=proto --go_out=proto/ proto/order.proto proto/trade.proto
```

### Service URLs
- Apollo API: http://localhost:8080
- Hermes WebSocket: http://localhost:8000
- PostgreSQL: localhost:5432
- Redis: localhost:6379
- Kafka: localhost:9092

## Code Structure

Each service follows the same pattern:
- `main.go`: Service entry point with graceful shutdown
- `database/`: Database models and queries
- `proto/`: Generated protobuf Go code (copied from root /proto)
- `env/`: Environment configuration
- Dockerfile for containerization

### Service-Specific Directories
- **Apollo**: `accounts/`, `users/`, `order/`, `product/`, `redis/`, `kafka/`, `middleware.go`
- **Hermes**: `hub.go`, `client.go`, `channels.go`, `consumer.go`, WebSocket message types
- **Matcher**: `engine/` (matching engine, orderbook, consumer/producer)

## Environment Configuration

Services use environment files in `/env/`:
- `kafka.env`: Kafka broker configuration  
- `postgres.env`: Database credentials
- `redis.env`: Redis configuration
- Individual service `.env` files

## Database

Initial schema is in `/db-init/tables.sql`. Database initialization happens automatically via Docker volumes when containers start.

## Prerequisites

- Docker and docker-compose
- protoc (Protocol Buffer compiler)
- Go 1.13+ (for local development)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/themarkrizkallah)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/themarkrizkallah)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
