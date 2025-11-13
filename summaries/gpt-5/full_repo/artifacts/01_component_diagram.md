```mermaid
graph TB
  %% Topology: Client -> MVC -> Controllers -> Repositories -> JPA -> DB; Controllers -> Thymeleaf Views
  subgraph Client
    Browser[Browser]
  end

  subgraph App["Spring Boot PetClinic (Monolith)"]
    Dispatcher["Spring MVC DispatcherServlet"]

    subgraph Web["Web Controllers"]
      OwnerCtrl["owner.OwnerController"]
      PetCtrl["owner.PetController"]
      VisitCtrl["owner.VisitController"]
      VetCtrl["vet.VetController"]
      WelcomeCtrl["system.WelcomeController"]
      CrashCtrl["system.CrashController"]
    end

    Views["Thymeleaf Views (templates/*.html)"]

    subgraph Infra["Cross-cutting"]
      Formatter["owner.PetTypeFormatter"]
      Validator["owner.PetValidator"]
      CacheCfg["system.CacheConfiguration (JCache)"]
      CacheVets["JCache 'vets' (Caffeine)"]
    end

    subgraph Data["Data Access"]
      OwnerRepo["owner.OwnerRepository (Spring Data JPA)"]
      VetRepo["vet.VetRepository (Spring Data JPA, @Cacheable)"]
      JPA["JPA/Hibernate EntityManager"]
    end

    subgraph Domain["Domain Model"]
      OwnerE["Owner"]
      PetE["Pet"]
      VisitE["Visit"]
      PetTypeE["PetType"]
      VetE["Vet"]
      SpecE["Specialty"]
    end
  end

  subgraph Persistence["Database"]
    DB[(H2 (default) | MySQL | Postgres)]
  end

  %% Client to MVC
  Browser -->|HTTP| Dispatcher

  %% MVC routing to controllers
  Dispatcher --> OwnerCtrl
  Dispatcher --> PetCtrl
  Dispatcher --> VisitCtrl
  Dispatcher --> VetCtrl
  Dispatcher --> WelcomeCtrl
  Dispatcher --> CrashCtrl

  %% Controllers to views
  OwnerCtrl -->|Render| Views
  PetCtrl -->|Render| Views
  VisitCtrl -->|Render| Views
  VetCtrl -->|Render| Views
  WelcomeCtrl -->|Render| Views
  CrashCtrl -->|Render| Views

  %% Controllers to repositories
  OwnerCtrl --> OwnerRepo
  PetCtrl --> OwnerRepo
  VisitCtrl --> OwnerRepo
  VetCtrl --> VetRepo

  %% Formatter/Validator participation
  PetCtrl -.->|Data binding| Formatter
  Dispatcher -.->|Conversion| Formatter
  Formatter -->|load types| OwnerRepo
  PetCtrl -.->|Validation| Validator

  %% Repositories to JPA/DB
  OwnerRepo --> JPA
  VetRepo -. "@Cacheable('vets')" .-> CacheVets
  CacheCfg --> CacheVets
  VetRepo -->|cache miss| JPA
  JPA --> DB

  %% Repositories manage domain aggregates
  OwnerRepo --> OwnerE
  OwnerRepo --> PetE
  OwnerRepo --> VisitE
  OwnerRepo --> PetTypeE
  VetRepo --> VetE
  VetRepo --> SpecE
```

The system is a monolithic Spring Boot MVC app: controllers handle HTTP requests and render Thymeleaf views (plus a JSON endpoint), directly invoking Spring Data JPA repositories without a separate service layer. VetRepository reads are cached via JCache/Caffeine configured in CacheConfiguration, while all persistence flows through JPA/Hibernate to H2/MySQL/Postgres (by profile). PetTypeFormatter and PetValidator integrate with Spring MVC binding/validation for form handling.