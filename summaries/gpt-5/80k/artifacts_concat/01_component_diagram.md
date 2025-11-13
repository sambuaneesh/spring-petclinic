```mermaid
graph TB
  Client[Browser / API Client]

  subgraph Application (Spring Boot Monolith)
    direction TB

    subgraph Bootstrap
      A1[PetClinicApplication]
      A2[PetClinicRuntimeHints]
    end

    subgraph Presentation (Spring MVC)
      I18N[LocaleChangeInterceptor (lang)]
      Views[Thymeleaf Views]
      WC[WelcomeController]
      CC[CrashController]
      VC[VetController]
      OC[OwnerController]
      PC[PetController]
      ViC[VisitController]
      Act[Actuator Endpoints]
    end

    subgraph Domain Model (JPA Aggregates)
      Owner[Owner (aggregate root)]
      Pet[Pet]
      Visit[Visit]
      PetType[PetType]
      Vet[Vet]
      Specialty[Specialty]
    end

    subgraph Persistence (Spring Data)
      OR[OwnerRepository]
      PTR[PetTypeRepository]
      VR[VetRepository (@Cacheable "vets")]
    end

    subgraph Cross-cutting
      CacheConfig[CacheConfiguration (JCache)]
      WebConfig[WebConfiguration (i18n)]
      PetVal[PetValidator]
      PetTypeFmt[PetTypeFormatter]
    end
  end

  subgraph Infrastructure
    DB[(Relational DB)]
    Cache[(JCache/Caffeine: "vets")]
  end

  %% Client interactions
  Client -->|GET /| WC
  Client -->|GET /oups| CC
  Client -->|/owners*| OC
  Client -->|/owners/{id}/pets*| PC
  Client -->|/owners/{id}/pets/{petId}/visits*| ViC
  Client -->|GET /vets.html| VC
  Client -->|GET /vets (JSON)| VC
  Client -->|/actuator/*| Act

  %% View rendering and JSON
  WC -->|HTML| Views
  OC -->|HTML| Views
  PC -->|HTML| Views
  ViC -->|HTML| Views
  VC -->|HTML vets.html| Views
  VC -.->|JSON response| Client

  %% Controllers -> Repositories
  OC --> OR
  PC --> OR
  PC --> PTR
  ViC --> OR
  VC --> VR

  %% Repositories -> DB
  OR --> DB
  PTR --> DB
  VR --> DB

  %% Vet cache
  VR -->|read-through| Cache
  CacheConfig --> Cache

  %% i18n and validation/formatting
  WebConfig --> I18N
  I18N -.-> WC
  I18N -.-> OC
  I18N -.-> PC
  I18N -.-> ViC
  I18N -.-> VC
  PetVal --> PC
  PetTypeFmt --> PC

  %% Domain relationships
  OR --> Owner
  PTR --> PetType
  VR --> Vet
  Owner -->|owns| Pet
  Pet -->|has| Visit
  Pet -->|type| PetType
  Vet -->|specialties| Specialty

  %% Bootstrap wiring
  A1 --> WebConfig
  A1 --> CacheConfig
  A1 --> Act
  A2 -.-> A1
```

The monolith is organized into presentation controllers that directly invoke Spring Data repositories without an intermediate service layer, operating on JPA aggregates (Owner→Pet→Visit) and persisting via cascading writes through OwnerRepository. The Veterinarian catalog is an independent slice using a read-through JCache-backed VetRepository with both HTML and JSON endpoints, while all interactions are synchronous and in-process apart from the shared relational database and cache; i18n is applied via a request interceptor.