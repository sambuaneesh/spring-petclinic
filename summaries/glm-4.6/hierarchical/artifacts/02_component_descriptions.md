
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies (frameworks, DBs, patterns) |
|---|---|---|---|---|
| Build Infrastructure | Ensures reproducible builds by managing Maven wrapper | MavenWrapperDownloader.main() | None | Maven, NIO, HTTP Basic Auth |
| Application Bootstrap | Application startup, configuration, and native image preparation | PetClinicApplication.main(), PetClinicRuntimeHints.registerHints() | All components | Spring Boot, GraalVM AOT, Maven |
| Model Layer | Defines core domain entities and validation rules | BaseEntity, Person, NamedEntity, ValidatorTests | None | JPA, Bean Validation, @MappedSuperclass |
| Service Layer | Orchestrates business logic and transactional workflows | EntityUtils.getById(), ClinicService methods | Model, Repositories | Spring, Generics, Transaction Management |
| Owner Management | Manages owners, pets, and veterinary visits | /owners/**, /pets/**, /visits/** endpoints | Model, Service | Spring MVC, Spring Data JPA, Thymeleaf, Pagination |
| Vet Management | Manages veterinarian information and specialties | /vets, /vets.html, VetRepository.findAll() | Model, Service | Spring MVC, Spring Data JPA, JCache, DTO Pattern |
| System Utilities | Provides cross-cutting concerns like caching and error handling | WelcomeController.welcome(), CrashController.triggerException() | None (cross-cutting) | Spring MVC, JCache, Exception Handling |