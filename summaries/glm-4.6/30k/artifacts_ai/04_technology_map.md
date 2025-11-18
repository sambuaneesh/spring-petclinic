
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|----------------|----------|------------|----------|---------------|----------|
| Owner Management | Java | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | HTTP REST endpoints (/owners) | MVC, Repository, Validation |
| Pet Management | Java | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | HTTP REST endpoints (/owners/*/pets) | MVC, Repository, Entity Relationships |
| Veterinary Management | Java | Spring Boot, Spring MVC, Spring Data JPA, Spring Cache | H2/MySQL/PostgreSQL | HTTP REST endpoints (/vets) | MVC, Repository, Caching, Many-to-Many |
| Visit Management | Java | Spring Boot, Spring MVC, Spring Data JPA | H2/MySQL/PostgreSQL | HTTP REST endpoints (/owners/*/pets/*/visits) | MVC, Repository, Entity Relationships |
| Data Access Layer | Java | Spring Data JPA, Hibernate | H2/MySQL/PostgreSQL | Repository interfaces | Repository, JPA Entities, Inheritance |
| Frontend/UI | HTML/JavaScript | Thymeleaf, Bootstrap 5, WebJars | N/A | Server-side rendering | Template Fragments, Internationalization |
| Validation | Java | Spring Validation, Bean Validation (JSR-380) | N/A | Form binding | Custom Validators, Annotation-based Validation |
| Configuration | Java | Spring Boot Configuration | N/A | Properties files | Profiles, Beans, Environment-specific configs |
| Caching | Java | Spring Cache, JCache, Caffeine | N/A | Method-level annotations | Cache-Aside, Read-through |
| Build & Deployment | XML/JSON/YAML | Maven, Gradle, Spring Boot Buildpacks | N/A | Docker registries | Containerization, Orchestration |
| Testing | Java | Spring Boot Test, MockMvc, Mockito, Testcontainers | H2/MySQL/PostgreSQL | HTTP mocks | Test Slicing, Integration Testing |
| Observability | Java | Spring Boot Actuator | N/A | JMX, HTTP endpoints | Health Checks, Metrics, Probes |
| Error Handling | Java | Spring Boot, Thymeleaf | N/A | Exception handling | Global Exception Handling |
| Internationalization | Java/Properties | Spring MVC, Thymeleaf | N/A | URL parameters | Message Bundles, Locale Resolution |