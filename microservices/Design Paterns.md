# Microservices Design Patterns

Here’s a concise but practical breakdown of common microservices patterns and when to use them, with real-world guidance for architects and developers.


## API Gateway Pattern

### What:
A single entry point for all clients that routes requests to appropriate microservices.

### Use When:
* You need centralized routing, auth, rate limiting, or versioning
* Want to hide internal services and reduce client-side complexity

### Tools:
AWS API Gateway, NGINX, Kong, Istio Ingress

## Service Discovery Pattern

### What:
Services automatically register themselves and discover each other without hard-coded addresses.

### Use When:
* You have dynamic scaling (containers, autoscaling)
* Services are short-lived (e.g., ECS, Kubernetes)

### Tools:
Eureka, Consul, AWS Cloud Map, Kubernetes DNS

## Strangler Fig Pattern
### What:
Gradually replace a monolith by routing specific calls to new microservices while leaving the rest untouched.

### Use When:
* Migrating legacy monoliths to microservices incrementally
* You need backward compatibility

### Tools:
API Gateway, ELB rules, feature flags, routing logic

## Database per Service Pattern
### What:
Each microservice has its own isolated database.

###  Use When:
* You want true decoupling
* Avoid cross-service transactions and improve autonomy

### Caveats:
Harder to do multi-service joins → Use events or API composition

## CQRS (Command Query Responsibility Segregation)
### What:
Separate models for read and write operations.

### Use When:
* You have high read/write imbalance
* Need event sourcing, audit logs, or complex queries

### Common in:
Banking systems, e-commerce carts, inventory

## Saga Pattern (Distributed Transactions)
### What:
Manages distributed transactions using compensating actions instead of 2PC.

### Use When:
* You need eventual consistency across services
* Example: booking system (reserve → pay → confirm)

### Variants:
* Choreography (event-driven)
* Orchestration (central controller)

## Event Sourcing Pattern
### What:
Instead of storing the current state, you store all events that led to it.

### Use When:
* You need auditability, replay, or temporal data
* Fits well with CQRS


## Bulkhead Pattern
### What:
Isolate parts of the system so a failure in one doesn't bring down others.

### Use When:
* You want to limit failure scope (e.g., isolate high-latency services)
* Think of it like fuses between components


## Circuit Breaker Pattern
### What:
Detects failures and short-circuits requests to failing services temporarily.

### Use When:
* Calling unreliable services (e.g., third-party APIs)
* Preventing cascading failures

### Tools:
Resilience4j, Hystrix (legacy), Istio retry policies


## Sidecar Pattern
### What:
Attach a sidecar container to a service for cross-cutting concerns (logging, metrics, TLS, etc.)

### Use When:
* Using service mesh
* Need observability or transparent retries/timeouts

### Tools:
Envoy, Istio, Linkerd

## API Composition Pattern
### What:
A composite service aggregates data from multiple services for the client.

### Use When:
* You need to avoid frontend over-fetching
* Clients depend on data from multiple microservices

## Backend for Frontend (BFF)
### What:
Each client type (web, mobile) gets its own backend tailored to its needs.

### Use When:
* You want optimized APIs for different UIs
* Avoid exposing internal APIs directly to the frontend