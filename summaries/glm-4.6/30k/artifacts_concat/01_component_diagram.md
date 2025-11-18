
```mermaid
graph TB
    subgraph "Web Layer"
        OwnerController[OwnerController]
        PetController[PetController]
        VisitController[VisitController]
        VetController[VetController]
        WelcomeController[WelcomeController]
        CrashController[CrashController]
    end
    
    subgraph "Business Logic & Validation"
        PetValidator[PetValidator]
        PetTypeFormatter[PetTypeFormatter]
        ClinicService[ClinicService]
    end
    
    subgraph "Data Access Layer"
        OwnerRepository[OwnerRepository]
        PetRepository[PetRepository]
        VisitRepository[VisitRepository]
        VetRepository[VetRepository]
        PetTypeRepository[PetTypeRepository]
    end
    
    subgraph "Domain Models"
        Owner[Owner]
        Pet[Pet]
        Visit[Visit]
        Vet[Vet]
        PetType[PetType]
        Specialty[Specialty]
    end
    
    subgraph "Infrastructure"
        Cache[CacheConfiguration]
        WebConfig[WebConfiguration]
        Thymeleaf[Thymeleaf Templates]
        Database[(Database)]
    end
    
    OwnerController --> OwnerRepository
    OwnerController --> PetValidator
    OwnerController --> PetTypeFormatter
    
    PetController --> PetRepository
    PetController --> OwnerRepository
    PetController --> PetValidator
    PetController --> PetTypeFormatter
    
    VisitController --> VisitRepository
    VisitController --> OwnerRepository
    VisitController --> PetRepository
    
    VetController --> VetRepository
    WelcomeController --> Thymeleaf
    CrashController --> Thymeleaf
    
    OwnerRepository --> Owner
    PetRepository --> Pet
    VisitRepository --> Visit
    VetRepository --> Vet
    PetTypeRepository --> PetType
    
    Owner --> Pet
    Pet --> Visit
    Pet --> PetType
    Vet --> Specialty
    
    VetRepository -.-> Cache
    OwnerController --> ClinicService
    PetController --> ClinicService
    VisitController --> ClinicService
    VetController --> ClinicService
    
    OwnerRepository --> Database
    PetRepository --> Database
    VisitRepository --> Database
    VetRepository --> Database
    PetTypeRepository --> Database
    
    WebConfig --> Thymeleaf
```

The architecture follows a traditional layered monolith pattern with clear separation between web controllers, business logic, data access, and domain models. Controllers directly interact with repositories following Spring's convention, with validation and formatting handled by dedicated components. The VetController demonstrates caching optimization while all repositories abstract database interactions, supporting multiple database backends through profile-based configuration.