| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On (other modules) | Technologies (frameworks, DBs, patterns) |
|---|---|---|---|---|
| PetClinicApplication | Bootstrap Spring Boot app | main(String[]); @SpringBootApplication | CacheConfiguration; WebConfiguration; Controllers; Repositories | Spring Boot |
| PetClinicRuntimeHints | Native image/AOT resource and serialization hints | registerHints(RuntimeHints, ClassLoader) | Domain model; message bundles; Spring AOT | Spring AOT; GraalVM native |
| CacheConfiguration | Enable and configure caching (cache “vets”) | @EnableCaching; cache settings via JCache | VetRepository | Spring Cache; JCache (javax.cache); Caffeine |
| WebConfiguration | MVC/i18n setup and interceptors | localeResolver(); addInterceptors(InterceptorRegistry) | Controllers; Thymeleaf views; message bundles | Spring MVC; i18n (SessionLocaleResolver; LocaleChangeInterceptor) |
| WelcomeController | Render home page | GET / -> welcome view | Thymeleaf views | Spring MVC; Thymeleaf |
| CrashController | Error-handling demo | GET /oups -> throws RuntimeException | Error view | Spring MVC; Spring Boot error handling |
| VetController | List veterinarians (HTML/JSON), pagination | GET /vets.html?page={n}; GET /vets (JSON) | VetRepository; CacheConfiguration; Thymeleaf views | Spring MVC; Spring Data Repository; Jackson; JCache/Caffeine |
| OwnerController | Owner CRUD, search, pagination | GET /owners/new; POST /owners/new; GET /owners/find; GET /owners; GET /owners/{ownerId}; GET/POST /owners/{ownerId}/edit | OwnerRepository; Validation; Thymeleaf views | Spring MVC; Spring Data JPA; Bean Validation; Pagination |
| PetController | Manage pets under owner; duplicate-name checks | GET/POST /owners/{ownerId}/pets/new; GET/POST /owners/{ownerId}/pets/{petId}/edit | OwnerRepository; PetTypeRepository; PetTypeFormatter; PetValidator; Thymeleaf | Spring MVC; Spring Data JPA; Bean Validation |
| VisitController | Create visits for a pet under an owner | GET/POST /owners/{ownerId}/pets/{petId}/visits/new | OwnerRepository; Thymeleaf; Validation | Spring MVC; Spring Data JPA; Bean Validation |
| OwnerRepository | Persistence for Owner aggregate (read/write) | findByLastNameStartingWith(String, Pageable); findById(Integer); save(Owner) | Owner entity | Spring Data JPA; JpaRepository |
| PetTypeRepository | Fetch PetType dictionary (sorted) | @Query findPetTypes(); findById(Integer); findAll() | PetType entity | Spring Data JPA; JpaRepository; JPQL |
| VetRepository | Read vets (cached), pagination | @Cacheable("vets") findAll(); @Cacheable("vets") findAll(Pageable) | Vet entity; CacheConfiguration | Spring Data Repository; JCache/Caffeine |
| PetTypeFormatter | UI conversion for PetType | parse(String, Locale); print(PetType, Locale) | PetTypeRepository | Spring Formatter SPI |
| PetValidator | Custom validation for Pet | supports(Class<?>); validate(Object, Errors) | Used by PetController | Spring Validation (Validator); Bean Validation (JSR 380) |
| Owner (Aggregate Root) | Owner data; manages Pets and Visits via cascade | addPet(Pet); getPet(name[,ignoreNew]); getPet(id); addVisit(petId, Visit) | Pet; Visit | JPA (@Entity); Aggregate Root; Cascade ALL; EAGER fetch; Bean Validation |
| Pet | Pet data; type and visits; add visit | addVisit(Visit); getters/setters | PetType; Visit; Owner (FK) | JPA; ManyToOne type; OneToMany visits (EAGER); Bean Validation |
| Visit | Visit details for a pet | getters/setters | Pet (FK) | JPA; Bean Validation (@NotBlank description) |
| Vet (Aggregate) | Vet data; specialties association; sorted specialties | getSpecialties(); addSpecialty(Specialty) | Specialty | JPA; ManyToMany (EAGER); NamedEntity |
| Specialty | Vet specialty lookup | getters/setters | — | JPA; NamedEntity |
| PetType | Pet type lookup | getters/setters | — | JPA; NamedEntity |
| Thymeleaf Views | Server-side rendered UI | Templates: welcome; owner create/edit/find/list/details; pet create/edit; visit create; vet list; error; fragments | Controllers; i18n | Thymeleaf; WebJars (Bootstrap, Font Awesome) |
| Actuator Exposure | Operational endpoints and probes | /actuator/*; /livez; /readyz | Application context; Health indicators | Spring Boot Actuator; health probes |
| Database Schema/Init | RDBMS schema and sample data | schema.sql; data.sql; profile-driven init | JPA; DataSource; application properties | RDBMS (H2, MySQL, PostgreSQL); Spring SQL init |
| Kubernetes Manifests | Deployment and service binding | k8s/petclinic.yml; k8s/db.yml (Service/Deployment/Secret) | Actuator (probes); Postgres; container image | Kubernetes; Service Binding spec |
| Build/CI Pipeline | Build, test, package, SBOM, native | Maven goals/plugins; GitHub Actions workflows | Entire project | Maven; spring-boot-maven-plugin; GraalVM plugin; Jacoco; Checkstyle; CycloneDX SBOM |