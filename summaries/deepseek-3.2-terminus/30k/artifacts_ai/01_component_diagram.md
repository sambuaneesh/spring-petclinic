```mermaid
graph TB
    WC[WelcomeController]
    OC[OwnerController]
    PC[PetController]
    VC[VisitController]
    VTC[VetController]
    CC[CrashController]
    
    CS[ClinicService]
    
    OR[OwnerRepository]
    PR[PetRepository]
    PTR[PetTypeRepository]
    VR[VetRepository]
    
    PV[PetValidator]
    PTF[PetTypeFormatter]
    
    DB[(Database)]
    
    WC --> CS
    OC --> OR
    OC --> PR
    PC --> OR
    PC --> PR
    PC --> PTR
    PC --> PV
    PC --> PTF
    VC --> OR
    VC --> PR
    VTC --> VR
    CC --> CS
    
    CS --> OR
    CS --> PTR
    CS --> VR
    
    OR --> DB
    PR --> DB
    PTR --> DB
    VR --> DB
    
    PV --> PTR
    PTF --> PTR
    
    CC -.->|Error Simulation| WC

```

The component boundaries follow Spring MVC layered architecture with clear separation between presentation (Controllers), business logic (Service/Validators), and data access (Repositories) layers. Communication patterns are primarily synchronous request-response through Spring's dependency injection, with controllers handling web requests and delegating to service/repository layers. The database acts as a central shared resource with repositories providing data access abstraction through Spring Data JPA.