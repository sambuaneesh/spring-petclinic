
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---|---|---|---|---|---|
| Owner Service | Java 17+ | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | REST | Repository Pattern, MVC, Pagination |
| Pet Service | Java 17+ | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | REST | Repository Pattern, MVC, Form Validation |
| Vet Service | Java 17+ | Spring Boot, Spring MVC, Spring Data JPA, Caffeine (JCache) | H2/MySQL/PostgreSQL | REST | Repository Pattern, MVC, Caching |
| Visit Service | Java 17+ | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | REST | Repository Pattern, MVC, Form Validation |
| Catalog Service | Java 17+ | Spring Boot, Spring Data JPA | H2/MySQL/PostgreSQL | REST | Repository Pattern, Caching |
| Frontend Layer | SCSS/CSS | Bootstrap, Thymeleaf | N/A | Server-Side Rendering (SSR) | Responsive Design, Mobile-First |
| Test Infrastructure | Java, JMeter config | JUnit 5, Testcontainers, Spring Test, JMeter | H2/MySQL/PostgreSQL (via containers) | N/A | Test Pyramid, Containerized Testing |
| Deployment & Ops | YAML, Shell | Docker, Kubernetes, GitHub Actions | MySQL, PostgreSQL (container images) | N/A | Infrastructure as Code, GitOps, Containerization |