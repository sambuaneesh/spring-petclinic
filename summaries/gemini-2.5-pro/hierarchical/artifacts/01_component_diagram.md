```markdown
graph TB
    subgraph UserFacing[Presentation Layer]
        OwnerController("owner.OwnerController")
        PetController("owner.PetController")
        VisitController("owner.VisitController")
        VetController("vet.VetController")
        WelcomeController("system.WelcomeController")
    end

    subgraph BusinessLogic[Service & Data Access Layer]
        direction LR
        subgraph Repositories
            OwnerRepo("owner.OwnerRepository")
            PetTypeRepo("owner.PetTypeRepository")
            VetRepo("vet.VetRepository")
        end
        subgraph BusinessValidation
            PetValidator("owner.PetValidator")
            ClinicServiceTests("service.ClinicServiceTests <br/><i>(Validates Logic)</i>")
        end
    end

    subgraph Core[Core & System Infrastructure]
        DomainModel("model.* <br/>(Owner, Pet, Vet, Visit)")
        SystemInfra("system.* <br/>(Caching, i18n, Error Handling)")
        AppCore("petclinic.PetClinicApplication <br/><i>(Bootstrap)</i>")
    end

    subgraph External
        DB[(Database)]
    end

    UserFacing -- "Uses for Data" --> BusinessLogic
    PetController -- "Uses for Validation" --> PetValidator
    BusinessLogic -- "Operates on" --> DomainModel
    BusinessLogic -- "Persists/Retrieves" --> DB
    
    OwnerController --> OwnerRepo
    PetController --> OwnerRepo
    PetController --> PetTypeRepo
    VisitController --> OwnerRepo
    VetController --> VetRepo

    SystemInfra -- "Enhances (e.g., Caching)" --> VetRepo
    SystemInfra -- "Supports (e.g., i18n, Errors)" --> UserFacing
    AppCore -- "Initializes" --> UserFacing
    AppCore -- "Initializes" --> BusinessLogic
    AppCore -- "Initializes" --> SystemInfra
```

### Rationale
The architecture is a classic layered system with a clear separation of concerns, primarily following the MVC and Repository patterns. The Presentation Layer, composed of Spring MVC controllers, handles user requests and orchestrates business operations by directly interacting with the Data Access Layer's repositories. A foundational Domain Model provides the core entities used across all layers, while a dedicated System component manages cross-cutting concerns like caching and internationalization that support the entire application.