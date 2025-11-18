
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---|---|---|---|---|---|
| Web Layer | Java 17 | Spring Boot 3.5.0, Spring MVC, Thymeleaf | N/A | HTTP/REST | MVC, RESTful APIs, Session Management |
| Data Access Layer | Java 17 | Spring Data JPA, Spring Boot, JCache | H2, MySQL 9.2, PostgreSQL 17.5 | N/A | Repository Pattern, Caching, ORM |
| Configuration Layer | Java 17 | Spring Boot, Spring Cache, Spring Web MVC | N/A | N/A | Configuration-as-Code, AOP, Dependency Injection |
| Testing Infrastructure | Java 17 | JUnit 5, Mockito, Testcontainers, Spring Test | Testcontainers (Docker) | MockMvc (HTTP) | Test Pyramid, Mocking, Container Testing |
| Deployment Infrastructure | N/A | Docker, Kubernetes | MySQL 9.2 (Docker) | HTTP, Container Networking | Containerization, Orchestration, Service Discovery |
| Owner Service | Java 17 | Spring Boot, Spring WebFlux, Spring Data | MySQL/PostgreSQL | REST, Events (Kafka/RabbitMQ) | Bounded Context, Saga, Event Sourcing |
| Pet Service | Java 17 | Spring Boot, Spring Data | PostgreSQL with JSON | REST, Events | Bounded Context, CQRS, Event-Driven |
| Visit Service | Java 17 | Spring Boot, Spring Data | PostgreSQL with TimescaleDB | REST, Events | Bounded Context, Time-Series, Event Sourcing |
| Vet Service | Java 17 | Spring Boot, Spring Data | PostgreSQL | REST, Events | Bounded Context, Read Replicas, Caching |
| API Gateway | Java 17 | Spring Cloud Gateway | N/A | HTTP/HTTPS | API Gateway, Circuit Breaker, Rate Limiting |
| Cache Layer | Java 17 | Spring Cache, Caffeine, JCache | N/A | N/A | Cache-Aside, Write-Through, Time-Based Expiration |
| Internationalization | Java 17 | Spring Boot, Thymeleaf | N/A | HTTP | Locale Resolution, Message Bundles, Session Storage |