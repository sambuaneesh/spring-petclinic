# Spring PetClinic Dynamic Interaction Flows

## Workflow 1: Owner Registration and Management

### Description
This workflow handles the complete lifecycle of owner management including registration, search, and profile updates. It's triggered when users need to register new pet owners or manage existing ones.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant OC as OwnerController
    participant OR as OwnerRepository
    participant DB as Database
    participant Val as Validator

    User->>OC: GET /owners/new
    OC-->>User: Render owner registration form
    
    User->>OC: POST /owners/new (form data)
    OC->>Val: Validate owner data
    Val-->>OC: Validation result
    
    alt Validation Failed
        OC-->>User: Return form with errors
    else Validation Passed
        OC->>OR: save(owner)
        OR->>DB: BEGIN TRANSACTION
        OR->>DB: INSERT INTO owners
        OR->>DB: COMMIT
        DB-->>OR: Owner ID
        OR-->>OC: Saved Owner entity
        OC-->>User: Redirect to /owners/{ownerId}
    end
    
    User->>OC: GET /owners/find
    OC-->>User: Render search form
    
    User->>OC: GET /owners?lastName=Smith&page=1
    OC->>OR: findByLastNameStartingWith("Smith", pageable)
    OR->>DB: SELECT * FROM owners WHERE last_name LIKE 'Smith%'
    DB-->>OR: Paginated results
    OR-->>OC: Page<Owner>
    OC-->>User: Render owner list
```

### Communication Patterns
- **Synchronous HTTP**: Form submissions and page navigation
- **Database Transactions**: ACID transactions for data consistency
- **Server-side Validation**: Bean validation with custom business rules
- **Pagination**: Database-level pagination for performance

---

## Workflow 2: Pet Registration and Medical History

### Description
This workflow manages pet registration, updates, and maintains the complete medical history through visit tracking. It's triggered when pets are registered or when medical visits occur.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant PC as PetController
    participant VC as VisitController
    participant OR as OwnerRepository
    participant PTR as PetTypeRepository
    participant Val as PetValidator
    participant DB as Database

    User->>PC: GET /owners/{ownerId}/pets/new
    PC->>OR: findById(ownerId)
    OR->>DB: SELECT * FROM owners WHERE id=ownerId
    DB-->>OR: Owner entity
    OR-->>PC: Owner
    PC->>PTR: findPetTypes()
    PTR->>DB: SELECT * FROM types
    DB-->>PTR: PetType list
    PTR-->>PC: Pet types
    PC-->>User: Render pet registration form
    
    User->>PC: POST /owners/{ownerId}/pets/new
    PC->>Val: Validate pet data
    Val->>Val: Check unique name per owner
    Val->>Val: Validate birth date not in future
    Val-->>PC: Validation result
    
    alt Validation Failed
        PC-->>User: Return form with errors
    else Validation Passed
        PC->>OR: save(ownerWithNewPet)
        OR->>DB: BEGIN TRANSACTION
        OR->>DB: INSERT INTO pets
        OR->>DB: UPDATE owners SET pets=...
        OR->>DB: COMMIT
        DB-->>OR: Success
        OR-->>PC: Updated Owner
        PC-->>User: Redirect to owner details
    end
    
    User->>VC: GET /owners/{ownerId}/pets/{petId}/visits/new
    VC-->>User: Render visit form
    
    User->>VC: POST /owners/{ownerId}/pets/{petId}/visits/new
    VC->>OR: findById(ownerId)
    OR->>DB: SELECT owner with pets and visits
    DB-->>OR: Owner with relationships
    OR-->>VC: Owner entity
    VC->>VC: Create new Visit with current date
    VC->>OR: save(ownerWithNewVisit)
    OR->>DB: INSERT INTO visits
    OR->>DB: UPDATE pets SET visits=...
    DB-->>OR: Success
    OR-->>VC: Updated Owner
    VC-->>User: Redirect to owner details
```

### Communication Patterns
- **Eager Loading**: Owner with pets and visits loaded in single query
- **Cascade Operations**: JPA cascade saves for related entities
- **Business Validation**: Custom validation rules for pet data
- **Transaction Management**: Atomic operations for data consistency

---

## Workflow 3: Veterinarian Directory Access

### Description
This workflow handles the display of veterinarian information with caching for performance optimization. It's triggered when users view the vet directory.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant VC as VetController
    participant VR as VetRepository
    participant Cache as JCache (vets)
    participant DB as Database

    User->>VC: GET /vets.html?page=1
    VC->>VR: findAll(pageable)
    VR->>Cache: Get "vets" cache entry
    
    alt Cache Hit
        Cache-->>VR: Cached Collection<Vet>
        VR-->>VC: Cached vets
    else Cache Miss
        VR->>DB: SELECT v.*, s.* FROM vets v<br/>LEFT JOIN vet_specialties vs ON v.id=vs.vet_id<br/>LEFT JOIN specialties s ON vs.specialty_id=s.id
        DB-->>VR: Vet data with specialties
        VR->>Cache: Put vets in cache
        VR-->>VC: Fresh vet data
    end
    
    VC-->>User: Render vet list (HTML)
    
    User->>VC: GET /vets (Accept: application/json)
    VC->>VR: findAll()
    Note over VR,Cache: Same cache lookup pattern
    VR-->>VC: Vet collection
    VC-->>User: Return JSON response
```

### Communication Patterns
- **Caching Strategy**: JCache with Caffeine for vet data
- **Eager Fetching**: Vets with specialties loaded together
- **Content Negotiation**: HTML and JSON responses from same endpoint
- **Cache-Aside Pattern**: Check cache first, then database

---

## Workflow 4: Error Handling and Internationalization

### Description
This workflow demonstrates the system's error handling capabilities and internationalization support across different user locales.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant WC as WelcomeController
    participant CC as CrashController
    participant WCfg as WebConfiguration
    participant DB as Database
    participant EH as ErrorHandler

    User->>WC: GET /?lang=es
    WCfg->>WCfg: Resolve locale from parameter "es"
    WCfg-->>User: Set session locale to Spanish
    WC-->>User: Render welcome page (Spanish)
    
    User->>CC: GET /oups
    CC->>CC: Simulate exception
    CC->>EH: Throw DataAccessException
    
    EH->>EH: Catch exception
    EH->>EH: Log error details
    EH->>EH: Determine error view based on exception type
    
    alt Database Connection Error
        EH-->>User: Render database error page
    else Validation Error
        EH-->>User: Render validation error with field details
    else General Error
        EH-->>User: Render generic error page
    end
    
    Note over User,EH: Error messages displayed in user's locale (Spanish)
```

### Communication Patterns
- **Locale Resolution**: Session-based and parameter-based locale switching
- **Exception Handling**: Spring MVC exception handlers with specific error views
- **Graceful Degradation**: User-friendly error pages instead of stack traces
- **Internationalization**: Error messages in user's preferred language

---

## Workflow 5: Search and Pagination Operations

### Description
This workflow handles search operations with pagination for large datasets, ensuring performance and user experience for data browsing.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as User (Browser)
    participant OC as OwnerController
    participant OR as OwnerRepository
    participant DB as Database
    participant Page as Pageable

    User->>OC: GET /owners?lastName=Smith&page=2
    OC->>Page: Create PageRequest(page=1, size=5)
    Page-->>OC: Pageable object
    
    OC->>OR: findByLastNameStartingWith("Smith", pageable)
    OR->>DB: SELECT * FROM owners<br/>WHERE last_name LIKE 'Smith%'<br/>LIMIT 5 OFFSET 5
    DB-->>OR: Page 2 results (records 6-10)
    
    OR->>DB: SELECT COUNT(*) FROM owners<br/>WHERE last_name LIKE 'Smith%'
    DB-->>OR: Total count
    
    OR->>OR: Create PageImpl with results and metadata
    OR-->>OC: Page<Owner> with pagination info
    
    OC->>OC: Prepare pagination model
    Note over OC: Calculate total pages, next/prev links
    OC-->>User: Render owner list with pagination controls
```

### Communication Patterns
- **Database Pagination**: LIMIT/OFFSET queries at database level
- **Count Queries**: Separate count queries for pagination metadata
- **Pageable Abstraction**: Spring Data pagination interface
- **Stateless Pagination**: URL parameters maintain pagination state

---

## Workflow 6: System Startup and Configuration

### Description
This workflow shows the application startup process including configuration loading, cache initialization, and database connection setup.

### Sequence Diagram
```mermaid
sequenceDiagram
    participant App as PetClinicApplication
    participant SC as Spring Context
    participant CC as CacheConfiguration
    participant WC as WebConfiguration
    participant DS as DataSource
    participant JPA as JPA/Hibernate
    participant DB as Database

    App->>SC: SpringApplication.run()
    SC->>CC: Initialize cache configuration
    CC->>CC: Create Caffeine cache manager
    CC->>CC: Configure "vets" cache with statistics
    CC-->>SC: CacheManager bean
    
    SC->>WC: Initialize web configuration
    WC->>WC: Set up locale resolvers
    WC->>WC: Configure message sources for i18n
    WC-->>SC: WebMvcConfigurer
    
    SC->>DS: Initialize DataSource
    DS->>DB: Establish connection
    DB-->>DS: Connection established
    
    SC->>JPA: Initialize JPA/Hibernate
    JPA->>DB: Validate database schema
    JPA->>DB: Check table existence
    DB-->>JPA: Schema valid
    
    SC->>SC: Initialize repositories and controllers
    SC->>SC: Scan for @Entity and @Repository
    SC->>SC: Wire dependencies
    
    SC-->>App: Application context ready
    App->>App: Start embedded Tomcat
    App-->>App: PetClinic application running on port 8080
```

### Communication Patterns
- **Dependency Injection**: Spring container wiring components
- **Lazy Initialization**: Components initialized on first use
- **Connection Pooling**: Database connection management
- **Configuration Profiles**: Environment-specific configuration loading