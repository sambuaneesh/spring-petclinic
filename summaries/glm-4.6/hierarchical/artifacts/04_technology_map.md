
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| Model Layer | Java | JPA, Bean Validation | Relational | None | Inheritance (MappedSuperclass), DRY |
| Service Layer | Java | Spring Framework | Relational (via repositories) | Internal | Service Layer, DRY, Test-Driven Development |
| Owner Management | Java | Spring MVC, Spring Data JPA | Relational | HTTP (REST) | MVC, Repository, Validation, Formatter |
| Veterinarian Management | Java | Spring MVC, Spring Data JPA | Relational | HTTP (REST) | Layered Architecture, DTO, Repository |
| System Infrastructure | Java | Spring MVC, JCache | None (caching) | HTTP (for web endpoints) | Separation of Concerns, Declarative Configuration |
| Application Bootstrap | Java | Spring Boot, Spring Test | None | None | Convention over Configuration, Test-Driven Development |
| Build Infrastructure | Java | None | None | HTTP | Bootstrap, Infrastructure as Code |