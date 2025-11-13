```markdown
```mermaid
graph TB
    subgraph "External"
        direction LR
        User[Web Browser]
        Client[REST Client]
    end

    subgraph "Spring PetClinic Monolith"
        direction TB

        subgraph "System Component"
            WelcomeController("WelcomeController")
            CrashController("CrashController")
            CacheConfig("CacheConfiguration")
        end

        subgraph "Owner Component"
            OwnerController("OwnerController") --> OwnerRepo("OwnerRepository")
            PetController("PetController") --> PetRepo("PetRepository")
            VisitController("VisitController") --> VisitRepo("VisitRepository")
            
            PetController -- "loads owner via" --> OwnerRepo
            VisitController -- "loads owner via" --> OwnerRepo
        end

        subgraph "Vet Component"
            VetController("VetController") --> VetRepo("VetRepository")
        end

        subgraph "Data Layer"
            Database[(Shared Database)]
        end
        
        %% External to Internal
        User -- "HTTP Requests" --> WelcomeController
        User -- "HTTP Requests" --> OwnerController
        User -- "HTTP Requests" --> PetController
        User -- "HTTP Requests" --> VisitController
        User -- "HTTP Requests" --> VetController
        Client -- "JSON API" --> VetController

        %% System Interactions
        CacheConfig -. "configures cache for" .-> VetController

        %% Data Access
        OwnerRepo --> Database
        PetRepo --> Database
        VisitRepo --> Database
        VetRepo --> Database
    end

    style User fill:#f9f,stroke:#333,stroke-width:2px
    style Client fill:#ccf,stroke:#333,stroke-width:2px
```

The diagram shows a classic monolithic architecture with component boundaries defined by logical business domains (`Owner`, `Vet`) and cross-cutting concerns (`System`). Communication is handled via in-process method calls, following a pattern where controllers directly interact with data repositories, bypassing a dedicated service layer. The single, shared database acts as the central point of data integration and coupling between all components.
```