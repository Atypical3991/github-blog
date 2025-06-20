# Architecture Patterns

There are several architecture patterns used in software system design, each suited for different use cases like scalability, maintainability, fault-tolerance, etc

## Layered (n-tier) Architecture
* Also called multi-tier architecture
* Organizes code into layers like:
    * Presentation (UI)
    * Business logic (Service)
    * Data access (Repository/DAO)

Use Case: Traditional enterprise apps, internal tools


## Client–Server Architecture
Divides system into clients (consumers) and servers (providers of data or services)

Use Case: Web apps, desktop apps communicating with backend

## Microservices Architecture
* Application is composed of small, independently deployable services
* Services communicate via REST, gRPC, Kafka, etc.

Use Case: Large-scale distributed systems, cloud-native apps

## Monolithic Architecture
All functionality in one tightly integrated application

Use Case: Small apps, MVPs, startups

## Event-Driven Architecture (EDA)
* Components communicate via events (async pub/sub model)
* Often built with message brokers like Kafka, RabbitMQ

Use Case: High-volume systems (e.g., e-commerce, streaming)

## Service-Oriented Architecture (SOA)
Like microservices but typically heavier, using SOAP/XML and ESBs

Use Case: Enterprise integrations, legacy systems

## Serverless Architecture
* Uses Function-as-a-Service (FaaS) (e.g., AWS Lambda)
* You deploy code without managing servers

Use Case: Trigger-based apps, lightweight APIs, automation

## Hexagonal Architecture (Ports and Adapters)
Isolates core logic from external systems (DB, web, messaging) using interfaces/adapters
Domain-driven design (DDD), clean architecture

## CQRS (Command Query Responsibility Segregation)
* Separates read and write models
* Often combined with Event Sourcing

Use Case: Systems with high read/write asymmetry, audit trail needs

## Peer-to-Peer (P2P)
No central server; each node acts as both client and server

Use Case: Blockchain, file sharing, torrents

# Choosing the Right Pattern

1. Fast delivery,small team: Monolithic, Layered
2. High scalability: Microservices, Serverless, Event-Driven
3. High complexity/domain-driven: Hexagonal, CQRS, Microservices
4. Asynchronous processing: Event-Driven, Message Bus
5. Low maintenance effort: Layered, MVC, Monolith




