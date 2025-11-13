```markdown
```mermaid
graph TB
  User[("User via Browser")]

  subgraph "Spring PetClinic Monolith"
    direction TB
    OwnerMgmt["Owner Management\n(Owners, Pets, Visits)"]
    VetMgmt["Veterinarian Management\n(Vets, Specialties)"]
    System["System & Cross-Cutting\n(Welcome, Errors, Caching)"]
  end

  Database[(Relational Database)]

  User -- "HTTP Requests" --> OwnerMgmt
  User -- "HTTP Requests" --> VetMgmt
  User -- "HTTP Requests" --> System

  OwnerMgmt -- "Reads/Writes Data (JPA)" --> Database
  VetMgmt -- "Reads Data (JPA)" --> Database

  System -. "Configures Caching for" .-> VetMgmt
```

The architecture is decomposed into three primary components reflecting the application's core domains: Owner Management (handling owners, pets, and visits), Veterinarian Management (vets and specialties), and a System component for cross-cutting concerns. As a monolith, all communication is via synchronous, in-process method calls, with both domain components sharing a single relational database for persistence and the System component providing configurations like caching to other components.
```