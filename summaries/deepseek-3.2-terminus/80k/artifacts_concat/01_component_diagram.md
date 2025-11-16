```mermaid
graph TB
    %% Domain Components
    OwnerController --> OwnerRepository
    OwnerController --> PetRepository
    PetController --> PetRepository
    PetController --> OwnerRepository
    PetController --> PetTypeRepository
    VisitController --> OwnerRepository
    VetController --> VetRepository
    
    %% Data Access Layer
    OwnerRepository --> Database[(Database)]
    PetRepository --> Database
    VetRepository --> Database
    PetTypeRepository --> Database
    
    %% Supporting Components
    WebConfiguration --> OwnerController
    WebConfiguration --> PetController
    WebConfiguration --> VisitController
    WebConfiguration --> VetController
    CacheConfiguration --> VetRepository
    
    %% External Dependencies
    OwnerController --> Thymeleaf[Templates]
    PetController --> Thymeleaf
    VisitController --> Thymeleaf
    VetController --> Thymeleaf
    
    %% Component Groups
    subgraph "Owner Management"
        OwnerController
        OwnerRepository
    end
    
    subgraph "Pet Management" 
        PetController
        PetRepository
        PetTypeRepository
    end
    
    subgraph "Veterinarian Management"
        VetController
        VetRepository
    end
    
    subgraph "Visit Management"
        VisitController
    end
    
    subgraph "Infrastructure"
        WebConfiguration
        CacheConfiguration
        Database
        Thymeleaf
    end
```

The component boundaries follow domain-driven design principles, with each domain aggregate (Owner, Pet, Vet, Visit) forming a natural cohesion boundary. Communication patterns are primarily synchronous HTTP requests from controllers to repositories, with the Vet component utilizing caching for performance optimization. The architecture maintains clear separation between web presentation, business logic, and data access layers while sharing a common database infrastructure.