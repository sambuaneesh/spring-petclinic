```mermaid
graph TB
  %% Top-level interaction
  Browser[Browser] -->|HTTP| Interceptors

  subgraph PetClinicApplication["Spring Boot Monolith (PetClinicApplication)"]
    subgraph WebLayer["Web Layer: Spring MVC + Thymeleaf"]
      Interceptors["LocaleChangeInterceptor + SessionLocaleResolver (i18n)"]
      Validators["Bean Validation + PetValidator + PetTypeFormatter"]

      Welcome[WelcomeController] --> Views
      Crash[CrashController]

      subgraph OwnerModule["Owners/Pets/Visits"]
        OwnerCtrl[OwnerController] --> Views
        PetCtrl[PetController] --> Views
        VisitCtrl[VisitController] --> Views
      end

      subgraph VetModule["Vets"]
        VetCtrl[VetController] --> VetView["vetList.html"]
        VetCtrl --> VetJson["/vets (JSON)"]
      end

      Actuator[Actuator (/actuator/**, /livez, /readyz)]
    end

    subgraph DomainPersistence["Domain + Persistence"]
      subgraph Entities["JPA Entities & Aggregates"]
        OwnerEntity["Owner (agg root)\n- pets EAGER\n- visits via Pet EAGER\n- cascade ALL"]
        PetEntity["Pet"]
        VisitEntity["Visit"]
        PetTypeEntity["PetType"]
        VetEntity["Vet"]
        SpecialtyEntity["Specialty"]
      end

      subgraph Repos["Spring Data Repositories"]
        OwnerRepo["OwnerRepository"]
        PetTypeRepo["PetTypeRepository"]
        VetRepo["VetRepository\n@Cacheable('vets')"]
      end

      JPA["Spring Data JPA / Hibernate"]
      Cache["JCache (Caffeine)\ncache: 'vets'"]
    end

    subgraph Config["Configuration & Boot"]
      WebConfig["WebConfiguration (i18n, interceptors)"]
      CacheConfig["CacheConfiguration (JCache)"]
      RuntimeHints["PetClinicRuntimeHints (AOT/Native)"]
    end
  end

  subgraph Datastore["Relational Database"]
    DB[(H2 / MySQL / PostgreSQL)]
  end

  %% Controller-to-repo calls (no service layer)
  OwnerCtrl --> OwnerRepo
  OwnerCtrl --> PetTypeRepo
  PetCtrl --> OwnerRepo
  PetCtrl --> PetTypeRepo
  VisitCtrl --> OwnerRepo
  VetCtrl --> VetRepo

  %% Validation usage
  OwnerCtrl -.-> Validators
  PetCtrl -.-> Validators
  VisitCtrl -.-> Validators

  %% Persistence flow
  OwnerRepo --> JPA
  PetTypeRepo --> JPA
  VetRepo --> JPA
  JPA --> DB

  %% Caching for vets
  VetRepo -->|@Cacheable| Cache

  %% General request flow
  Interceptors --> Welcome
  Interceptors --> OwnerCtrl
  Interceptors --> PetCtrl
  Interceptors --> VisitCtrl
  Interceptors --> VetCtrl
  Interceptors --> Actuator
```

Rationale:
- The monolith follows classic MVC: controllers directly invoke Spring Data repositories (no service layer), rendering Thymeleaf views for HTML and exposing a single JSON endpoint (/vets). All domain entities persist via a single relational database through JPA/Hibernate, with the Owner aggregate cascading writes to Pets and Visits.
- Cross-cutting concerns include i18n via a locale interceptor and a read-through JCache (Caffeine) on VetRepository; Actuator exposes operational endpoints. Communication is entirely in-process and synchronous, with server-side rendering for most endpoints.