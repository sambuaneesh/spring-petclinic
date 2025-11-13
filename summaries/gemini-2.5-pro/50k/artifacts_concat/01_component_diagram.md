```mermaid
graph TB
    User[("User")]
    ApiClient[("API Client")]

    subgraph "Spring PetClinic Monolith"
        UI["WebApp UI (Thymeleaf)"]
        OwnerComponent["Owner Component\n(Owner, Pet, Visit Management)"]
        VetComponent["Vet Component\n(Veterinarian Management)"]
        SystemComponent["System Component\n(Welcome, Error, Config)"]
        SharedModel["Shared Data Models\n(Person, BaseEntity)"]
    end

    subgraph "External Services"
        DB[(Relational Database)]
        Cache[(JCache)]
    end

    User -- HTTP Requests --> UI

    UI --> OwnerComponent
    UI --> VetComponent
    UI --> SystemComponent

    ApiClient -- JSON/XML API --> VetComponent

    OwnerComponent -- CRUD Operations --> DB
    VetComponent -- CRUD Operations --> DB
    VetComponent -- Caches Vet List --> Cache

    OwnerComponent -. uses .-> SharedModel
    VetComponent -. uses .-> SharedModel
```

The architecture is a classic monolith where component boundaries are logically defined by business domains (`Owner` and `Vet`) within the same deployment unit. Communication is primarily synchronous and in-process, with a central UI layer delegating requests to the appropriate domain component. While both components share a common database and data model, the `Vet` component is more decoupled, utilizing its own cache and exposing an independent data API for external clients.