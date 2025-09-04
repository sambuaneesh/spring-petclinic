```mermaid
graph TB
    subgraph "User Interface (Thymeleaf)"
        WebUI[Web UI]
    end

    subgraph "Application Components"
        subgraph "Owner Management"
            OwnerController["OwnerController, PetController, VisitController"]
            OwnerRepositories["OwnerRepository, PetTypeRepository"]
        end

        subgraph "Veterinarian Management"
            VetController[VetController]
            VetRepository[VetRepository]
        end

        subgraph "System"
            SystemControllers[WelcomeController, CrashController]
            SystemConfig[WebConfiguration, CacheConfiguration]
        end
    end

    subgraph "Shared Kernel"
        SharedModel[Shared Domain Model <br/> (BaseEntity, Person, etc.)]
    end

    subgraph "Data Persistence"
        Database[(Shared Relational Database <br/> H2 / MySQL / PostgreSQL)]
    end

    %% Interactions
    WebUI --> OwnerController
    WebUI --> VetController
    WebUI --> SystemControllers

    OwnerController --> OwnerRepositories
    VetController --> VetRepository

    OwnerRepositories --> SharedModel
    VetRepository --> SharedModel

    OwnerRepositories --> Database
    VetRepository --> Database

    SystemConfig -- "Applies Caching" --> VetRepository
    SystemConfig -- "Applies i18n" --> WebUI
```

### Rationale

The architecture is a modular monolith, with component boundaries drawn along clear domain lines: "Owner Management" for all customer-facing concerns and "Veterinarian Management" for staff-related information. These components communicate internally through direct method calls and share a common data model and a single database, which is characteristic of a monolithic design. Cross-cutting concerns like routing, caching, and internationalization are handled by a central "System" component that configures behavior across the application.