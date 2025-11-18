
```mermaid
graph TB
    %% Frontend Layer
    subgraph "Presentation Layer"
        UI[Thymeleaf Templates<br/>Bootstrap UI]
        WelcomeCtrl[WelcomeController<br/>/]
        OwnerCtrl[OwnerController<br/>/owners]
        PetCtrl[PetController<br/>/owners/{id}/pets]
        VisitCtrl[VisitController<br/>/owners/{id}/pets/{pid}/visits]
        VetCtrl[VetController<br/>/vets]
        CrashCtrl[CrashController<br/>/oups]
    end

    %% Business Logic Layer
    subgraph "Service Layer"
        OwnerSvc[Owner Service<br/>CRUD + Search]
        PetSvc[Pet Service<br/>Management]
        VisitSvc[Visit Service<br/>Scheduling]
        VetSvc[Vet Service<br/>Specialties]
        CatalogSvc[Catalog Service<br/>PetTypes & Specialties]
    end

    %% Data Access Layer
    subgraph "Repository Layer"
        OwnerRepo[OwnerRepository]
        PetRepo[PetRepository]
        VisitRepo[VisitRepository]
        VetRepo[VetRepository<br/>@Cacheable]
        PetTypeRepo[PetTypeRepository]
        SpecialtyRepo[SpecialtyRepository]
    end

    %% Cross-cutting Concerns
    subgraph "Infrastructure"
        Validation[Validation Layer<br/>Jakarta Validation]
        I18N[Internationalization<br/>messages.properties]
        Cache[Cache Layer<br/>Caffeine/JCache]
        Monitoring[Actuator<br/>Health Checks]
    end

    %% Data Store
    subgraph "Database Layer"
        DB[(Database<br/>H2/MySQL/PostgreSQL)]
    end

    %% Connections - Controllers to Services
    WelcomeCtrl --> UI
    OwnerCtrl --> OwnerSvc
    OwnerCtrl --> UI
    PetCtrl --> PetSvc
    PetCtrl --> UI
    VisitCtrl --> VisitSvc
    VisitCtrl --> UI
    VetCtrl --> VetSvc
    VetCtrl --> UI
    CrashCtrl --> UI

    %% Connections - Services to Repositories
    OwnerSvc --> OwnerRepo
    PetSvc --> PetRepo
    PetSvc --> PetTypeRepo
    VisitSvc --> VisitRepo
    VisitSvc --> OwnerRepo
    VetSvc --> VetRepo
    VetSvc --> SpecialtyRepo
    CatalogSvc --> PetTypeRepo
    CatalogSvc --> SpecialtyRepo

    %% Connections - Repositories to Database
    OwnerRepo --> DB
    PetRepo --> DB
    VisitRepo --> DB
    VetRepo --> DB
    PetTypeRepo --> DB
    SpecialtyRepo --> DB

    %% Cross-cutting concerns integration
    Validation --> OwnerCtrl
    Validation --> PetCtrl
    Validation --> VisitCtrl
    Validation --> OwnerSvc
    Validation --> PetSvc
    Validation --> VisitSvc
    
    I18N --> UI
    Cache --> VetRepo
    Monitoring --> DB
```