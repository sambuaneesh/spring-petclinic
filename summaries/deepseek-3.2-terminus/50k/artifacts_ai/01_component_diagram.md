```mermaid
graph TB
    %% Presentation Layer
    WelcomeController[WelcomeController]
    CrashController[CrashController]
    ThymeleafTemplates[Thymeleaf Templates]
    
    %% Domain Components
    OwnerController[OwnerController]
    OwnerRepository[OwnerRepository]
    OwnerEntity[Owner Entity]
    
    PetController[PetController]
    PetRepository[PetRepository]
    PetEntity[Pet Entity]
    PetValidator[PetValidator]
    
    VetController[VetController]
    VetRepository[VetRepository]
    VetEntity[Vet Entity]
    
    VisitController[VisitController]
    VisitEntity[Visit Entity]
    
    %% System Components
    CacheConfiguration[CacheConfiguration]
    WebConfiguration[WebConfiguration]
    Database[(Database)]
    
    %% Frontend
    Frontend[Frontend Assets]
    
    %% Interactions
    WelcomeController --> ThymeleafTemplates
    CrashController --> ThymeleafTemplates
    
    OwnerController --> OwnerRepository
    OwnerController --> ThymeleafTemplates
    OwnerRepository --> OwnerEntity
    OwnerRepository --> Database
    
    PetController --> PetRepository
    PetController --> PetValidator
    PetController --> ThymeleafTemplates
    PetRepository --> PetEntity
    PetRepository --> Database
    
    VetController --> VetRepository
    VetController --> ThymeleafTemplates
    VetRepository --> VetEntity
    VetRepository --> Database
    VetRepository --> CacheConfiguration
    
    VisitController --> ThymeleafTemplates
    VisitController --> Database
    
    ThymeleafTemplates --> Frontend
    WebConfiguration --> ThymeleafTemplates
    
    %% Entity Relationships
    OwnerEntity --> PetEntity
    PetEntity --> VisitEntity
    VetEntity --> SpecialtyEntity[Specialty Entity]
```

The component boundaries follow domain-driven design principles with clear separation between presentation, business logic, and data access layers. Communication patterns are primarily synchronous request-response via Spring MVC controllers, with repository abstractions handling database interactions. Cross-cutting concerns like caching and internationalization are managed through centralized configuration components that integrate with the domain layers.