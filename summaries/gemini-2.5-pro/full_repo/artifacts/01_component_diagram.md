```markdown
graph TB
    subgraph "Spring PetClinic Application (Monolith)"
        direction TB
        User[Browser User] --> WebLayer{Web Layer / UI}

        subgraph "Owner Management"
            OwnerController["OwnerController"]
            PetController["PetController"]
            VisitController["VisitController"]
            OwnerRepo["OwnerRepository"]
            PetTypeRepo["PetTypeRepository"]
        end

        subgraph "Veterinarian Management"
            VetController["VetController"]
            VetRepo["VetRepository"]
        end

        subgraph "System & Configuration"
            SystemControllers["Welcome/Crash Controllers"]
            CacheConfig["CacheConfiguration"]
        end

        WebLayer --> OwnerController
        WebLayer --> PetController
        WebLayer --> VisitController
        WebLayer --> VetController
        WebLayer --> SystemControllers

        OwnerController --> OwnerRepo
        PetController --> OwnerRepo
        PetController --> PetTypeRepo
        VisitController --> OwnerRepo
        VetController --> VetRepo

        CacheConfig -- "Configures Caching for" --> VetRepo
    end

    subgraph "Data Persistence"
      Database[(Relational DB<br/>H2 / MySQL / PostgreSQL)]
    end

    OwnerRepo -- "CRUD via JPA" --> Database
    PetTypeRepo -- "CRUD via JPA" --> Database
    VetRepo -- "CRUD via JPA" --> Database
```

The system is a classic Spring Boot monolith structured with a layered architecture and organized by domain features. Component boundaries are defined by Java packages (`owner`, `vet`, `system`), each encapsulating controllers, domain models, and repositories for a specific business capability like managing owners or vets. Communication is primarily internal via method calls, where Spring MVC controllers handle HTTP requests and delegate business logic and data persistence operations to Spring Data JPA repositories.