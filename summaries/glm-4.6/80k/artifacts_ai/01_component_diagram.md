
```mermaid
graph TB
    subgraph "Web Layer"
        OwnerController["OwnerController<br/>(/owners)"]
        PetController["PetController<br/>(/owners/{id}/pets)"]
        VisitController["VisitController<br/>(/owners/{id}/pets/{id}/visits)"]
        VetController["VetController<br/>(/vets)"]
        SystemController["SystemController<br/>(/, /oups)"]
    end

    subgraph "Service Layer"
        OwnerService["OwnerService"]
        PetService["PetService"]
        VisitService["VisitService"]
        VetService["VetService"]
    end

    subgraph "Repository Layer"
        OwnerRepo["OwnerRepository"]
        PetTypeRepo["PetTypeRepository"]
        VisitRepo["VisitRepository"]
        VetRepo["VetRepository"]
        SpecialtyRepo["SpecialtyRepository"]
    end

    subgraph "Domain Model"
        Owner["Owner<br/>(extends Person)"]
        Pet["Pet<br/>(extends NamedEntity)"]
        Visit["Visit<br/>(extends BaseEntity)"]
        Vet["Vet<br/>(extends Person)"]
        PetType["PetType<br/>(extends NamedEntity)"]
        Specialty["Specialty<br/>(extends NamedEntity)"]
    end

    subgraph "External Systems"
        Database[(Database<br/>H2/MySQL/PostgreSQL)]
        Cache[(Cache<br/>Caffeine/JCache)]
        Thymeleaf["Thymeleaf<br/>Templates"]
    end

    subgraph "Configuration"
        WebConfig["WebConfiguration<br/>(Locale/i18n)"]
        CacheConfig["CacheConfiguration"]
        RuntimeConfig["RuntimeHintsConfiguration<br/>(GraalVM)"]
    end

    %% Controller to Service interactions
    OwnerController --> OwnerService
    PetController --> PetService
    VisitController --> VisitService
    VetController --> VetService
    SystemController -.-> OwnerService

    %% Service to Repository interactions
    OwnerService --> OwnerRepo
    PetService --> PetTypeRepo
    VisitService --> VisitRepo
    VetService --> VetRepo
    VetService --> SpecialtyRepo

    %% Repository to Domain interactions
    OwnerRepo --> Owner
    PetTypeRepo --> PetType
    VisitRepo --> Visit
    VetRepo --> Vet
    SpecialtyRepo --> Specialty

    %% Repository to Database
    OwnerRepo --> Database
    PetTypeRepo --> Database
    VisitRepo --> Database
    VetRepo --> Database
    SpecialtyRepo --> Database

    %% Controller to View
    OwnerController --> Thymeleaf
    PetController --> Thymeleaf
    VisitController --> Thymeleaf
    VetController --> Thymeleaf
    SystemController --> Thymeleaf

    %% Cache interactions
    PetTypeRepo --> Cache
    VetRepo --> Cache
    CacheConfig --> Cache

    %% Configuration to Components
    WebConfig --> OwnerController
    WebConfig --> VetController
    CacheConfig --> PetTypeRepo
    CacheConfig --> VetRepo
    RuntimeConfig -.-> Owner
    RuntimeConfig -.-> PetType

    %% Domain relationships
    Owner --> Pet
    Pet --> Visit
    Pet --> PetType
    Vet --> Specialty
```

The component boundaries follow a classic layered architecture pattern with clear separation of concerns: Controllers handle web requests, Services encapsulate business logic, Repositories manage data access, and Domain Models represent core entities. Communication flows primarily top-down through synchronous method calls, with caching implemented at the repository layer for performance optimization and configuration classes providing cross-cutting concerns like internationalization and GraalVM support.