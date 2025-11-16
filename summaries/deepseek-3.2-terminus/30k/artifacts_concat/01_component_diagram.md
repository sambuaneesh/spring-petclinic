```mermaid
graph TB
    %% Web Layer Components
    subgraph WebLayer [Web Layer]
        WelcomeController[WelcomeController]
        OwnerController[OwnerController]
        PetController[PetController]
        VisitController[VisitController]
        VetController[VetController]
        CrashController[CrashController]
    end

    %% Business Logic Layer
    subgraph BusinessLayer [Business Logic Layer]
        ClinicService[ClinicService]
        PetValidator[PetValidator]
        PetTypeFormatter[PetTypeFormatter]
    end

    %% Data Access Layer
    subgraph DataLayer [Data Access Layer]
        OwnerRepository[OwnerRepository]
        PetTypeRepository[PetTypeRepository]
        VetRepository[VetRepository]
        EntityUtils[EntityUtils]
    end

    %% Domain Model
    subgraph DomainLayer [Domain Model]
        Owner[Owner Entity]
        Pet[Pet Entity]
        Visit[Visit Entity]
        Vet[Vet Entity]
        PetType[PetType Entity]
        Specialty[Specialty Entity]
    end

    %% External Systems
    subgraph ExternalLayer [External Systems]
        Database[(Database)]
        Cache[Caffeine Cache]
        Templates[Thymeleaf Templates]
        WebJars[WebJars Resources]
    end

    %% Web Layer Interactions
    WelcomeController --> Templates
    OwnerController --> ClinicService
    OwnerController --> Templates
    PetController --> ClinicService
    PetController --> PetValidator
    PetController --> Templates
    VisitController --> ClinicService
    VisitController --> Templates
    VetController --> VetRepository
    VetController --> Templates
    VetController --> Cache
    CrashController --> Templates

    %% Business Layer Interactions
    ClinicService --> OwnerRepository
    ClinicService --> PetTypeRepository
    ClinicService --> VetRepository
    PetValidator --> DomainLayer
    PetTypeFormatter --> PetTypeRepository

    %% Data Layer Interactions
    OwnerRepository --> Database
    OwnerRepository --> Owner
    PetTypeRepository --> Database
    PetTypeRepository --> PetType
    VetRepository --> Database
    VetRepository --> Vet
    VetRepository --> Cache
    EntityUtils --> DomainLayer

    %% Domain Relationships
    Owner --> Pet
    Pet --> Visit
    Pet --> PetType
    Vet --> Specialty

    %% External Dependencies
    Templates --> WebJars
```

The component boundaries follow a traditional layered architecture with clear separation between web, business logic, data access, and domain layers. Communication patterns are primarily synchronous and request-driven, with controllers handling web requests, services coordinating business logic, and repositories managing data persistence. The architecture demonstrates strong domain cohesion within each layer while maintaining loose coupling through dependency injection and interface-based abstractions.