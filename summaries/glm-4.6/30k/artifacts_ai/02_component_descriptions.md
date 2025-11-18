
| Component Name | Responsibility | Interfaces | Depends On | Technologies |
|----------------|----------------|------------|------------|--------------|
| OwnerController | Owner CRUD operations, search, form handling | GET/POST `/owners/find`, `/owners`, `/owners/{ownerId}`, `/owners/new`, `/owners/{ownerId}/edit` | OwnerRepository, PetRepository, VisitRepository | Spring MVC, Thymeleaf, Bean Validation |
| PetController | Pet management within owner context | GET/POST `/owners/{ownerId}/pets/new`, `/owners/{ownerId}/pets/{petId}/edit` | PetRepository, OwnerRepository, PetTypeRepository | Spring MVC, PetValidator, Bean Validation |
| VisitController | Visit scheduling and management | GET/POST `/owners/{ownerId}/pets/{petId}/visits/new` | OwnerRepository, VisitRepository | Spring MVC, Thymeleaf |
| VetController | Veterinary staff listing with caching | GET `/`, GET `/vets.html`, GET `/vets` (JSON) | VetRepository | Spring MVC, JCache/Caffeine, Pagination |
| CrashController | Error handling and exception testing | GET `/oups` | None | Spring MVC |
| OwnerRepository | Owner data access with custom queries | `findByLastNameStartingWith()`, `findById()` | JPA/Hibernate | Spring Data JPA |
| PetRepository | Pet data access | `findById()` | JPA/Hibernate | Spring Data JPA |
| VisitRepository | Visit data access | `findByPetId()` | JPA/Hibernate | Spring Data JPA |
| VetRepository | Veterinarian data access with caching | `findAll()`, `findAll(Pageable)` | JPA/Hibernate | Spring Data JPA, JCache |
| SpecialtyRepository | Specialty data access | Standard CRUD operations | JPA/Hibernate | Spring Data JPA |
| PetTypeRepository | Pet type data access | `findPetTypes()` | JPA/Hibernate | Spring Data JPA |
| Entity Layer | Domain model definitions with relationships and validation | JPA annotations, validation annotations | None | JPA/Hibernate, JSR-380 Validation, Serializable |
| CacheConfiguration | Caching setup and statistics management | CacheConfiguration bean | Spring Boot | JCache API, Caffeine |
| WebConfiguration | Internationalization, locale handling, formatters | LocaleChangeInterceptor, PetTypeFormatter | Spring MVC | Spring Boot, Thymeleaf |
| Database Layer | Multi-database persistence with schema management | JDBC, JPA | Entity Layer | H2/MySQL/PostgreSQL, Spring Data JPA |
| Frontend Layer | UI rendering and user interaction | Thymeleaf templates, fragments | Spring MVC controllers | Thymeleaf, Bootstrap 5, WebJars |
| Testing Layer | Application quality assurance and validation | @WebMvcTest, @DataJpaTest test classes | Application components | Spring Boot Test, MockMvc, Mockito, Testcontainers |
| ActuatorLayer | Application monitoring and health checks | `/health`, `/info`, `/metrics`, `/livez`, `/readyz` | Spring Boot | Spring Boot Actuator, JMX |
| BuildSystem | Application compilation, packaging, and deployment | Maven/Gradle commands, Docker builds | Source code | Maven/Gradle, Spring Boot Plugin, Docker, Testcontainers |