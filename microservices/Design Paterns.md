# Microservices Design Patterns

## Architectural style
Which architectural style should you choose for an application?
* Monolithic Architecture: architect an application as a single deployable unit
* Microservices Architecture: architect an application as a collection of independently deployable, loosely coupled services

## Decomposition Patterns
### Decomposition by business capabilities
Define services corresponding to business capabilities. A business capability is a concept from business architecture modeling. It is something that a business does in order to generate value. A business capability often corresponds to a business object, e.g.
* Order Management is responsible for orders
* Customer Management is responsible for customers

### Decompose by Subdomain (DDD)
Decompose using Domain-Driven Design bounded contexts.

The subdomains of an online store include:
* User Management in the Customer context.
* Pricing Engine in the Sales context.

## Integration Patterns
### API Gateway
Central entry point for clients.

For e.g. Zuul, Kong, or AWS API Gateway routes /api/orders to Order Service.

### Aggregator Pattern
A service composes responses from multiple microservices.

For e.g.
ProductDetailsService calls InventoryService, PriceService, and ReviewService and combines the results.

## Database Patterns
### Database per Service
Each microservice owns its schema.
For e.g. 
* OrderService → orders_db
* UserService → users_db

### Shared Database (Anti-pattern)
Only used temporarily when refactoring monoliths.
Note: Can create tight coupling and scalability issues.

## Communication Patterns
### Synchronous REST/HTTP
For e.g. OrderService calls PaymentService via REST during checkout.
### Asynchronous Messaging (Event-Driven)
For e.g. OrderService publishes OrderCreated → InventoryService and EmailService consume the event.

## Resiliency Patterns
### Circuit Breaker
For e.g. If PaymentService is down, the Circuit Breaker (like Netflix Hystrix) prevents repeated failures from affecting OrderService.
### Retry
For e.g. Retry calling a failed external API up to 3 times with exponential backoff.
### Bulkhead
For e.g. Limit concurrent requests to a service to isolate failure. Usage of connection pooling pods.

## Observability Patterns
### Log Aggregation
For e.g. Use ELK Stack or Grafana Loki to aggregate logs from all services.
### Distributed Tracing
For e.g. Use Jaeger or Zipkin to trace a request from API Gateway → OrderService → PaymentService.

## Data Consistency Patterns
### Saga Pattern
For e.g. 
For order placement:
* OrderService creates an order.
* Sends event → InventoryService reserves stock.
* → PaymentService charges payment.
* If any step fails, compensating transactions are executed.

### Event Sourcing
Persist state changes as events.
For e.g. OrderService stores OrderPlaced, OrderConfirmed events and reconstructs state from them.


## Security Patterns
### Access Token Propagation
For e.g. JWT token from client is propagated to all services for authorization.

### Service-to-Service Authentication
For e.g. Use mTLS or service mesh like Istio for secure internal communication. Authentication gets handled at mesh level.

## Deployment Patterns
### Service per Host / Container
For e.g. OrderService runs in its own Docker container or Kubernetes Pod.
### Serverless Functions
For e.g. Use AWS Lambda for EmailNotificationService.

## UI Patterns
### Backend for Frontend (BFF)
Create a dedicated API for each frontend (mobile/web).
For e.g. MobileBFF for mobile clients, WebBFF for web apps.

