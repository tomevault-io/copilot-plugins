## micro-commerce

> Construir uma plataforma completa de e-commerce utilizando arquitetura moderna baseada em Microservices.

# AGENTS.md

# MicroCommerce

## Objetivo

Construir uma plataforma completa de e-commerce utilizando arquitetura moderna baseada em Microservices.

O projeto será utilizado como portfólio profissional e deverá demonstrar competências avançadas em:

* Microservices
* DDD
* Clean Architecture
* CQRS
* Event Driven Architecture
* Saga Pattern
* Outbox Pattern
* RabbitMQ
* Redis
* PostgreSQL
* MongoDB
* Docker
* Kubernetes
* OpenTelemetry
* Prometheus
* Grafana
* API Gateway
* BFF Pattern
* Next.js

O projeto deve priorizar:

* Qualidade arquitetural
* Clareza de implementação
* Documentação
* Boas práticas
* Testabilidade

---

# Arquitetura Geral

```text
Browser
   |
Next.js Frontend
   |
Web BFF
   |
API Gateway
   |
-----------------------------------------------------
|         |          |          |         |          |
Auth   Customer   Catalog     Cart      Order   Inventory
                                          |
                                       RabbitMQ
                                          |
                         ----------------------------
                         |            |             |
                      Payment   Notification     Audit
```

---

# Estrutura do Repositório

```text
microcommerce/

src/

├── frontend/
│   └── web/
│
├── bff/
│   └── web-bff/
│
├── gateway/
│   └── api-gateway/
│
├── services/
│   ├── auth-service/
│   ├── customer-service/
│   ├── catalog-service/
│   ├── cart-service/
│   ├── order-service/
│   ├── inventory-service/
│   ├── payment-service/
│   ├── notification-service/
│   └── audit-service/
│
├── building-blocks/
│   ├── shared-kernel/
│   ├── contracts/
│   ├── event-bus/
│   ├── observability/
│   └── outbox/
│
├── tests/
│
├── docs/
│
├── docker/
│
└── k8s/
```

---

# Regras Gerais

## Obrigatório

Todos os serviços devem possuir:

* Dockerfile
* Swagger
* Health Check
* Structured Logging
* OpenTelemetry
* Unit Tests
* Integration Tests

---

## Proibido

Não compartilhar banco entre serviços.

Não compartilhar entidades entre serviços.

Não realizar consultas SQL entre domínios.

Não utilizar comunicação direta via banco.

Toda integração entre domínios deve ocorrer por:

* API
* Eventos

---

# Backend Stack

## Framework

* .NET 8

## APIs

* ASP.NET Core
* Minimal APIs

## Arquitetura

* DDD
* Clean Architecture
* CQRS
* MediatR
* FluentValidation

## Resiliência

* Polly
* Retry
* Circuit Breaker
* Timeout

---

# Frontend Stack

## Framework

* Next.js 15

## Linguagem

* TypeScript

## UI

* Tailwind CSS

## Estado Global

* Zustand

## Server State

* TanStack Query

## HTTP

* Axios

---

# BFF Stack

## Framework

* .NET 8

## APIs

* Minimal APIs

## Responsabilidades

* Agregação de endpoints
* Orquestração de chamadas
* Cache
* View Models
* Autenticação
* Redução de chamadas do frontend

---

# Banco de Dados

## PostgreSQL

Utilizar para:

* Auth Service
* Customer Service
* Catalog Service
* Order Service
* Inventory Service

---

## MongoDB

Utilizar para:

* Payment Service
* Audit Service

---

## Redis

Utilizar para:

* Cart Service
* Cache Distribuído
* Sessões

---

# Mensageria

## RabbitMQ

Utilizar para:

* Integração entre domínios
* Saga Pattern
* Eventos de negócio

---

# Shared Kernel

Localização:

```text
building-blocks/shared-kernel
```

Implementar:

* Entity
* AggregateRoot
* ValueObject
* Result
* DomainEvent

---

# Contracts

Localização:

```text
building-blocks/contracts
```

Implementar:

* Integration Events
* Event Metadata
* CorrelationId
* Event Version

Todos os eventos devem ser imutáveis.

Utilizar record.

Exemplo:

```csharp
public record OrderCreatedEvent(
    Guid EventId,
    Guid CorrelationId,
    DateTime OccurredAt,
    int Version,
    Guid OrderId,
    Guid CustomerId,
    decimal TotalAmount
);
```

---

# API Gateway

Responsável por:

* Roteamento
* JWT Validation
* Rate Limiting
* Observabilidade

Tecnologia:

* YARP

---

# Frontend

## Funcionalidades

### Home

* Produtos em destaque
* Categorias

### Catálogo

* Listagem de produtos
* Busca
* Filtro

### Produto

* Detalhes
* Adicionar ao carrinho

### Carrinho

* Adicionar item
* Remover item
* Atualizar quantidade

### Checkout

* Finalizar compra

### Conta

* Login
* Cadastro

### Pedidos

* Histórico de pedidos

---

# BFF

## Endpoints

### Catálogo

```http
GET /catalog
GET /catalog/{id}
```

### Carrinho

```http
GET /cart
POST /cart
DELETE /cart
```

### Pedido

```http
POST /checkout
GET /orders
GET /orders/{id}
```

### Usuário

```http
POST /login
POST /register
```

O frontend nunca deve consumir diretamente os microservices.

Toda comunicação deve passar pelo BFF.

---

# Microservices

## Auth Service

Responsabilidades:

* Registro
* Login
* JWT
* Refresh Token

Banco:

PostgreSQL

---

## Customer Service

Responsabilidades:

* Clientes
* Endereços

Banco:

PostgreSQL

---

## Catalog Service

Responsabilidades:

* Produtos
* Categorias

Banco:

PostgreSQL

---

## Cart Service

Responsabilidades:

* Carrinho

Banco:

Redis

---

## Order Service

Responsabilidades:

* Pedidos

Banco:

PostgreSQL

Eventos:

* OrderCreated
* OrderConfirmed
* OrderCancelled

---

## Inventory Service

Responsabilidades:

* Estoque
* Reserva

Banco:

PostgreSQL

Eventos:

* InventoryReserved
* InventoryReleased
* InventoryFailed

---

## Payment Service

Responsabilidades:

* Processamento de pagamentos

Banco:

MongoDB

Eventos:

* PaymentApproved
* PaymentRejected

---

## Notification Service

Responsabilidades:

* Notificações

Consumidor de eventos.

---

## Audit Service

Responsabilidades:

* Auditoria

Consumidor de todos os eventos.

Banco:

MongoDB

---

# CQRS

Todos os serviços devem separar:

## Commands

Exemplos:

```text
CreateOrderCommand
CreateCustomerCommand
CreateProductCommand
```

## Queries

Exemplos:

```text
GetOrderByIdQuery
GetCustomerQuery
GetProductsQuery
```

---

# Saga Pattern

Fluxo principal:

```text
OrderCreated
      |
InventoryReserved
      |
PaymentApproved
      |
OrderConfirmed
```

Fluxo de compensação:

```text
PaymentRejected
      |
InventoryReleased
      |
OrderCancelled
```

Utilizar Saga Orquestrada.

---

# Outbox Pattern

Obrigatório.

Nenhum evento pode ser publicado diretamente após SaveChanges.

Fluxo:

```text
Save Aggregate
      |
Save Outbox Event
      |
Commit Transaction
      |
Background Dispatcher
      |
RabbitMQ
```

---

# Observabilidade

Todos os serviços devem expor:

```http
GET /health

GET /metrics
```

Instrumentar:

* HTTP
* RabbitMQ
* PostgreSQL
* MongoDB
* Redis

Utilizar:

* OpenTelemetry
* Prometheus
* Grafana

---

# Docker

Todos os projetos devem possuir Dockerfile.

Deve existir docker-compose.yml contendo:

* Frontend
* BFF
* Gateway
* RabbitMQ
* PostgreSQL
* MongoDB
* Redis
* Grafana
* Prometheus

Comando esperado:

```bash
docker compose up -d
```

---

# Kubernetes

Criar manifests para:

* Deployments
* Services
* ConfigMaps
* Secrets
* Ingress

Estrutura:

```text
k8s/

frontend/
bff/
gateway/

auth-service/
customer-service/
catalog-service/
cart-service/
order-service/
inventory-service/
payment-service/
notification-service/
audit-service/
```

---

# Testes

Cobertura mínima:

```text
80%
```

Implementar:

## Unit Tests

* Domain
* Application

## Integration Tests

* PostgreSQL
* MongoDB
* Redis
* RabbitMQ

## End-to-End Tests

Fluxos:

* Login
* Cadastro
* Compra
* Checkout
* Consulta de pedidos

---

# Critério de Sucesso

O projeto será considerado concluído quando:

* Frontend funcional.
* BFF funcional.
* Gateway funcional.
* Todos os microservices independentes.
* RabbitMQ funcionando.
* Saga Pattern implementada.
* Outbox Pattern implementada.
* Observabilidade funcionando.
* Docker Compose funcional.
* Kubernetes funcional.
* Testes automatizados passando.
* Documentação completa.
* Projeto apto para demonstração em entrevistas técnicas.

---
> Source: [Sameque/micro-commerce](https://github.com/Sameque/micro-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
