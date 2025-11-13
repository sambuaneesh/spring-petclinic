```mermaid
graph TB
  U[Browser / API Client]

  subgraph "PetClinic Monolith (Spring Boot)"
    direction TB

    subgraph "Presentation (Spring MVC)"
      OC[OwnerController]
      PC[PetController]
      VC[VisitController]
      VTC[VetController]
      WC[WelcomeController]
      CC[CrashController]
    end

    subgraph "Views & Static"
      TPL[Thymeleaf HTML Views]
      JSON[JSON/XML endpoint (/vets)]
      STATIC[Static Assets (CSS/JS via WebJars)]
    end

    subgraph "Validation & Binding"
      PV[PetValidator]
      PTF[PetTypeFormatter]
    end

    subgraph "Domain Model (JPA Entities)"
      OwnerE[Owner]
      PetE[Pet]
      VisitE[Visit]
      PetTypeE[PetType]
      VetE[Vet]
      SpecE[Specialty]
    end

    subgraph "Data Access (Spring Data JPA)"
      OR[OwnerRepository]
      PTR[PetTypeRepository]
      VR[VetRepository]
    end

    subgraph "Cross-cutting"
      subgraph "Caching (JCache + Caffeine)"
        VetsCache[Cache: vets]
      end
      WConf[WebConfiguration\n(LocaleResolver + Interceptor)]
      Act[Actuator (/actuator/*, /livez, /readyz)]
    end
  end

  DB[(Relational DB\nH2 / MySQL / PostgreSQL)]

  %% Client interactions
  U --> OC
  U --> PC
  U --> VC
  U --> VTC
  U --> WC
  U --> CC
  U --> Act
  U --> TPL
  U --> STATIC

  %% Controller -> view/api
  OC --> TPL
  PC --> TPL
  VC --> TPL
  WC --> TPL
  CC --> TPL
  VTC --> TPL
  VTC --> JSON

  %% Validation & formatting
  PC --> PV
  PC --> PTF

  %% Controller -> repositories
  OC --> OR
  PC --> OR
  PC --> PTR
  VC --> OR
  VTC --> VR

  %% Repositories -> DB
  OR --> DB
  PTR --> DB
  VR --> DB

  %% Caching on vets
  VR <-->|"@Cacheable('vets')"| VetsCache

  %% I18n/Web config (representative)
  WConf -.-> OC

  %% Controllers/Repos use domain entities (representative)
  OC -.-> OwnerE
  PC -.-> PetE
  VC -.-> VisitE
  VTC -.-> VetE
  OR -.-> OwnerE
  PTR -.-> PetTypeE
  VR -.-> VetE
```

The monolith follows a classic layered architecture: MVC controllers render Thymeleaf HTML (and one JSON endpoint) and invoke Spring Data repositories directly, with validation/formatting applied in the web layer. Data access uses JPA to a single relational database; the vets read path is optimized with a local JCache/Caffeine cache. Cross-cutting i18n configuration and Actuator endpoints apply framework-level concerns, and there are no inter-service calls—clients and the database are the only external interactions.