
```mermaid
graph TB
    subgraph "Web Layer"
        OwnerController[OwnerController<br/>Owner/Pet Management]
        VisitController[VisitController<br/>Visit Scheduling]
        VetController[VetController<br/>Veterinary Staff]
        WelcomeController[WelcomeController<br/>Home Page]
        CrashController[CrashController<br/>Error Handling]
    end
    
    subgraph "Data Access Layer"
        OwnerRepository[OwnerRepository<br/>Search/Find Owners]
        PetTypeRepository[PetTypeRepository<br/>Pet Type Catalog]
        VetRepository[VetRepository<br/>Vet Directory<br/>Cached]
    end
    
    subgraph "Database Layer"
        DB[(Database<br/>H2/MySQL/PostgreSQL)]
    end
    
    subgraph "Cross-Cutting Concerns"
        Cache[JCache/Caffeine<br/>Caching Layer]
        I18N[Internationalization<br/>8 Languages]
    end
    
    %% Web Layer interactions
    OwnerController --> OwnerRepository
    OwnerController --> PetTypeRepository
    VisitController --> OwnerRepository
    VetController --> VetRepository
    WelcomeController --> VetController
    
    %% Repository to Database
    OwnerRepository --> DB
    PetTypeRepository --> DB
    VetRepository --> DB
    
    %% Cross-cutting concerns
    VetRepository -.-> Cache
    OwnerController -.-> I18N
    VetController -.-> I18N
    WelcomeController -.-> I18N
    CrashController -.-> I18N
```

The architecture follows a classic three-tier pattern with clear separation between presentation, data access, and persistence layers. Controllers handle web requests and orchestrate business flows through repositories, which abstract database operations. Cross-cutting concerns like caching and internationalization are implemented as orthogonal services that integrate across multiple layers without tight coupling.