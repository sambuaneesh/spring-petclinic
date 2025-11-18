
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies |
|----------------|----------------|----------------------------------------|------------|--------------|
| OwnerController | Handle owner-related web requests and form processing | GET/POST /owners/new, GET/POST /owners/find, GET /owners, GET/POST /owners/{id}/edit, GET /owners/{id} | OwnerRepository, PetService | Spring MVC, Thymeleaf, Validation |
| VetController | Manage veterinarian listing and display | GET /vets.html, GET /vets | VetRepository, SpecialtyRepository | Spring MVC, Thymeleaf, Caching |
| VisitController | Handle visit scheduling for pets | GET/POST /owners/{ownerId}/pets/{petId}/visits/new | VisitRepository, PetRepository | Spring MVC, Thymeleaf |
| PetController | Manage pet operations within owner context | GET/POST /owners/{ownerId}/pets/new, GET/POST /owners/{ownerId}/pets/{petId}/edit | PetRepository, PetTypeRepository, OwnerRepository | Spring MVC, Thymeleaf |
| OwnerRepository | Data access for owner entities with search capabilities | findByLastNameStartingWith(), findById(), save(), delete() | Database (H2/MySQL/PostgreSQL) | Spring Data JPA, Pagination |
| VetRepository | Data access for veterinarian information | findAll(), findAll(Pageable) | Database (H2/MySQL/PostgreSQL) | Spring Data JPA, JCache |
| PetTypeRepository | Manage pet type classifications with caching | findPetTypes() | Database (H2/MySQL/PostgreSQL) | Spring Data JPA, Caffeine Cache |
| SpecialtyRepository | Handle veterinary specialty data | findAll(), findById() | Database (H2/MySQL/PostgreSQL) | Spring Data JPA |
| VisitRepository | Data access for visit records | findByPetId(), save(), findAll() | Database (H2/MySQL/PostgreSQL) | Spring Data JPA |
| CacheConfiguration | Configure application-level caching with Caffeine | cacheManager() bean | Caffeine library | Spring Cache, JCache, Caffeine |
| WebConfiguration | Configure web layer settings including internationalization | localeResolver(), localeChangeInterceptor() | Spring Framework | Spring MVC, Thymeleaf |
| RuntimeHintsConfiguration | Provide GraalVM native image compilation hints | registerHints() | GraalVM | Spring Boot Native |
| Domain Models | Define business entities and relationships | Owner, Pet, Visit, Vet, Specialty, PetType | JPA/Hibernate | Spring Data, Validation |
| Database Schema | Physical data storage with relational integrity | owners, pets, visits, vets, specialties, vet_specialties tables | H2/MySQL/PostgreSQL | SQL, Foreign Keys, Indexes |
| MessageSource | Handle internationalization and localization | messages_*.properties bundles | Spring Framework | ResourceBundle, UTF-8 |