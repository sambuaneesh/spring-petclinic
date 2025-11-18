
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On (other modules) | Technologies (frameworks, DBs, patterns) |
|---|---|---|---|---|
| OwnerController | Handle HTTP requests for owner management operations (CRUD) | GET /owners/find, GET /owners, GET /owners/{ownerId}, GET /owners/new, POST /owners/new, GET /owners/{ownerId}/edit, POST /owners/{ownerId}/edit | OwnerRepository | Spring MVC, Thymeleaf |
| PetController | Handle HTTP requests for pet management operations (CRUD) | GET /owners/{ownerId}/pets/new, POST /owners/{ownerId}/pets/new, GET /owners/{ownerId}/pets/{petId}/edit, POST /owners/{ownerId}/pets/{petId}/edit | OwnerRepository, PetRepository, PetTypeRepository | Spring MVC, Thymeleaf |
| VisitController | Handle HTTP requests for visit management operations (Create) | GET /owners/{ownerId}/pets/{petId}/visits/new, POST /owners/{ownerId}/pets/{petId}/visits/new | OwnerRepository, PetRepository, VisitRepository | Spring MVC, Thymeleaf |
| VetController | Handle HTTP requests for veterinary operations (listing) | GET /, GET /vets.html, GET /vets | VetRepository | Spring MVC, Thymeleaf |
| WelcomeController | Handle root path request | GET / | None | Spring MVC, Thymeleaf |
| CrashController | Handle error scenarios | GET /oups | None | Spring MVC |
| OwnerRepository | Data access layer for Owner entities | findByLastNameStartingWith(String, Pageable), findById(Integer) | JPA/Hibernate, Database | Spring Data JPA |
| PetRepository | Data access layer for Pet entities | Standard CRUD operations | JPA/Hibernate, Database | Spring Data JPA |
| VisitRepository | Data access layer for Visit entities | Standard CRUD operations | JPA/Hibernate, Database | Spring Data JPA |
| VetRepository | Data access layer for Vet entities with caching | findAll(), findAll(Pageable) | JPA/Hibernate, Database | Spring Data JPA, JCache with Caffeine |
| PetTypeRepository | Data access layer for PetType entities | findPetTypes() | JPA/Hibernate, Database | Spring Data JPA |
| BaseEntity | Base entity with auto-generated ID | getId(), setId() | None | JPA/Hibernate |
| NamedEntity | Entity with name field | getName(), setName() | BaseEntity | JPA/Hibernate |
| Person | Entity with person fields (first name, last name) | getFirstName(), setFirstName(), getLastName(), setLastName() | BaseEntity | JPA/Hibernate, Bean Validation |
| Owner | Represents pet owners with contact info | getAddress(), setAddress(), getCity(), setCity(), getTelephone(), setTelephone(), getPets() | Person | JPA/Hibernate, Bean Validation |
| Pet | Represents pets with type, birth date, and visits | getBirthDate(), setBirthDate(), getType(), setType(), getVisits() | NamedEntity | JPA/Hibernate, Bean Validation |
| Visit | Represents vet visits with date and description | getVisitDate(), setVisitDate(), getDescription(), setDescription() | BaseEntity | JPA/Hibernate, Bean Validation |
| PetType | Represents pet categories | getName(), setName() | NamedEntity | JPA/Hibernate |
| Vet | Represents veterinarians with specialties | getSpecialties(), setSpecialties() | Person | JPA/Hibernate |
| Specialty | Represents veterinary specialties | getName(), setName() | NamedEntity | JPA/Hibernate |
| Vets | Wrapper for vet collections for serialization | getVetList(), setVetList() | List<Vet> | Jackson (XML/JSON serialization) |
| PetValidator | Validates pet-specific constraints | validate(Object, Errors) | Spring Validation | Spring Validation |
| PetTypeFormatter | Converts strings to PetType and vice versa | print(PetType, Locale), parse(String, Locale) | PetTypeRepository | Spring Formatter |
| CacheConfiguration | Configures caching behavior | N/A | JCache, Caffeine | JCache, Caffeine |
| WebConfiguration | Configures web layer including i18n | N/A | Spring MVC | Spring MVC, Thymeleaf, Message Bundles |
| EntityUtils | Utility for entity lookup by ID in collections | getById(Collection, int, Class) | None | Java Collections |
| VisitControllerTests | Tests VisitController endpoints | testInitNewVisitForm(), testProcessNewVisitForm() | OwnerRepository, MockMvc | Spring Boot Test, Mockito |
| VetControllerTests | Tests VetController endpoints | testShowVetList(), testShowResourcesVetList() | VetRepository, MockMvc | Spring Boot Test, Mockito |
| CrashControllerTests | Tests error handling scenarios | testTriggerException() | None | Spring Boot Test |
| ClinicServiceTests | Integration tests for service and repository layers | testFindOwners(), testInsertPet(), etc. | JPA/Hibernate, Database | Spring Boot Test, Testcontainers |
| I18nPropertiesSyncTest | Validates internationalization properties synchronization | validateProperties() | File System | Spring Boot Test, JUnit |