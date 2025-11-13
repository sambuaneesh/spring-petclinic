```mermaid
graph TB

  Client[Browser / API Client]

  subgraph Presentation / MVC
    WelcomeController[WelcomeController (/)]
    OwnerController[OwnerController (/owners...)]
    PetController[PetController (/owners/{ownerId}/pets...)]
    VisitController[VisitController (/owners/{ownerId}/pets/{petId}/visits...)]
    VetController[VetController (/vets.html, /vets JSON)]
    CrashController[CrashController (/oups)]
    Thymeleaf[Thymeleaf Views/Templates]
    VetsDTO[Vets DTO (JSON/XML wrapper)]
  end

  subgraph Infrastructure / Cross-cutting
    PetClinicApplication[PetClinicApplication (Spring Boot)]
    RuntimeHints[PetClinicRuntimeHints (AOT/native)]
    WebConfiguration[WebConfiguration (i18n, LocaleChangeInterceptor)]
    CacheConfiguration[CacheConfiguration (@EnableCaching)]
    PetTypeFormatter[PetTypeFormatter]
    PetValidator[PetValidator]
    Actuator[Actuator (/livez, /readyz, /actuator/*)]
  end

  subgraph Domain - Owner Aggregate
    OwnerEntity[Owner]
    PetEntity[Pet]
    VisitEntity[Visit]
    PetTypeEntity[PetType]
  end

  subgraph Domain - Vet
    VetEntity[Vet]
    SpecialtyEntity[Specialty]
  end

  subgraph Persistence
    OwnerRepository[OwnerRepository]
    PetTypeRepository[PetTypeRepository]
    VetRepository[VetRepository @Cacheable("vets")]
    JPA[Spring Data JPA / Hibernate]
    Cache[JCache (Caffeine) - vets]
    DB[(RDBMS: H2 / MySQL / Postgres)]
  end

  %% Entry and UI flow
  PetClinicApplication --> WelcomeController
  PetClinicApplication --> OwnerController
  PetClinicApplication --> PetController
  PetClinicApplication --> VisitController
  PetClinicApplication --> VetController
  PetClinicApplication --> CrashController
  RuntimeHints -. resource/serialization hints .-> PetClinicApplication

  Client --> WelcomeController
  Client --> OwnerController
  Client --> PetController
  Client --> VisitController
  Client --> VetController
  Client --> CrashController
  Client --> Actuator

  WelcomeController --> Thymeleaf
  OwnerController --> Thymeleaf
  PetController --> Thymeleaf
  VisitController --> Thymeleaf
  VetController --> Thymeleaf
  VetController --> VetsDTO

  %% Controllers -> Repositories
  OwnerController --> OwnerRepository
  PetController --> OwnerRepository
  PetController --> PetTypeRepository
  VisitController --> OwnerRepository
  VetController --> VetRepository

  %% Repositories -> Persistence
  OwnerRepository --> JPA
  PetTypeRepository --> JPA
  VetRepository --> JPA
  JPA --> DB

  %% Caching
  CacheConfiguration --> Cache
  VetRepository -- "@Cacheable('vets')" --> Cache

  %% i18n / validation / formatting
  WebConfiguration -. locales & interceptor .-> Presentation / MVC
  PetTypeFormatter -. type parsing/printing .-> PetController
  PetValidator -. validation .-> PetController

  %% Repository entity scopes
  OwnerRepository --> OwnerEntity
  PetTypeRepository --> PetTypeEntity
  VetRepository --> VetEntity

  %% Domain relationships
  OwnerEntity -- "cascade=ALL, fetch=EAGER" --> PetEntity
  PetEntity -- "cascade=ALL, fetch=EAGER" --> VisitEntity
  PetEntity --> PetTypeEntity
  VetEntity --- SpecialtyEntity
```

Rationale:
The monolith follows a layered MVC architecture: controllers render Thymeleaf views (and JSON for /vets) and call Spring Data repositories, which delegate to JPA/Hibernate against a single relational database. The Owner aggregate encapsulates Pets and Visits with cascaded, eager relationships, while the Vet domain is relatively independent and read-optimized via a JCache/Caffeine-backed cache. Cross-cutting infrastructure provides i18n, validation/formatting, actuator probes, and AOT hints; there is no inter-service communication.