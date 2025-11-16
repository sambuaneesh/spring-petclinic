```markdown
# Spring PetClinic Dynamic Interaction Flows

## Workflow 1: Owner Registration and Pet Management

### Description
Complete workflow for registering a new owner, adding pets, and scheduling veterinary visits. This represents the core customer journey through the application.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant PC as PetController
    participant VC as VisitController
    participant OR as OwnerRepository
    participant PR as PetRepository
    participant VR as VisitRepository
    participant PT as PetTypeRepository
    participant PV as PetValidator
    participant DB as Database

    Note over U,DB: Owner Registration Phase
    U->>OC: GET /owners/new
    OC-->>U: Render owner creation form
    
    U->>OC: POST /owners/new (owner data)
    OC->>PV: Validate owner data
    PV-->>OC: Validation result
    OC->>OR: save(owner)
    OR->>DB: INSERT INTO owners
    DB-->>OR: Owner ID
    OR-->>OC: Saved owner entity
    OC-->>U: Redirect to owner details

    Note over U,DB: Pet Registration Phase
    U->>PC: GET /owners/{id}/pets/new
    PC->>OR: findById(ownerId)
    OR->>DB: SELECT owner by ID
    DB-->>OR: Owner data
    OR-->>PC: Owner entity
    PC->>PT: findAll()
    PT->>DB: SELECT pet_types
    DB-->>PT: Pet types
    PT-->>PC: Pet type list
    PC-->>U: Render pet creation form

    U->>PC: POST /owners/{id}/pets/new (pet data)
    PC->>PV: Validate pet data
    PV->>PR: Check pet name uniqueness
    PR->>DB: SELECT pets by owner
    DB-->>PR: Existing pets
    PR-->>PV: Validation data
    PV-->>PC: Validation result
    PC->>PR: save(pet)
    PR->>DB: INSERT INTO pets
    DB-->>PR: Pet ID
    PR-->>PC: Saved pet entity
    PC-->>U: Redirect to owner details

    Note over U,DB: Visit Scheduling Phase
    U->>VC: GET /owners/{oid}/pets/{pid}/visits/new
    VC->>OR: findById(ownerId)
    OR->>DB: SELECT owner by ID
    DB-->>OR: Owner data
    OR-->>VC: Owner entity
    VC-->>U: Render visit scheduling form

    U->>VC: POST /owners/{oid}/pets/{pid}/visits/new (visit data)
    VC->>VR: save(visit)
    VR->>DB: INSERT INTO visits
    DB-->>VR: Visit ID
    VR-->>VC: Saved visit entity
    VC-->>U: Redirect to owner details
```

### Communication Patterns
- **Synchronous HTTP**: Form submissions and redirects
- **Database Transactions**: JPA-managed transactions for data consistency
- **Server-Side Validation**: Bean validation and custom business rules
- **Repository Pattern**: Data access abstraction through Spring Data JPA

---

## Workflow 2: Owner Search and Profile Management

### Description
Workflow for searching owners by last name with pagination and managing owner profiles through edit operations.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant OR as OwnerRepository
    participant DB as Database

    Note over U,DB: Owner Search Phase
    U->>OC: GET /owners/find
    OC-->>U: Render search form
    
    U->>OC: GET /owners?lastName=Smith&page=1
    OC->>OR: findByLastNameStartingWith("Smith", PageRequest)
    OR->>DB: SELECT owners WHERE lastName LIKE 'Smith%'
    DB-->>OR: Paginated owner results
    OR-->>OC: Page<Owner> object
    OC-->>U: Render search results with pagination

    Note over U,DB: Owner Profile View
    U->>OC: GET /owners/{ownerId}
    OC->>OR: findById(ownerId)
    OR->>DB: SELECT owner with pets and visits (EAGER fetch)
    DB-->>OR: Complete owner graph
    OR-->>OC: Owner with relationships
    OC-->>U: Render owner profile with pets/visits

    Note over U,DB: Owner Edit Phase
    U->>OC: GET /owners/{ownerId}/edit
    OC->>OR: findById(ownerId)
    OR->>DB: SELECT owner data
    DB-->>OR: Owner entity
    OR-->>OC: Owner for editing
    OC-->>U: Render edit form

    U->>OC: POST /owners/{ownerId}/edit (updated data)
    OC->>PV: Validate updated owner data
    PV-->>OC: Validation result
    OC->>OR: save(updatedOwner)
    OR->>DB: UPDATE owners
    DB-->>OR: Updated record
    OR-->>OC: Saved owner entity
    OC-->>U: Redirect to owner details
```

### Communication Patterns
- **Pagination**: Database-level pagination with Spring Data
- **Eager Fetching**: Owner-pet-visit relationship loading
- **Form Validation**: Server-side validation before persistence
- **Optimistic Locking**: JPA versioning for concurrent updates

---

## Workflow 3: Veterinarian Directory with Caching

### Description
High-performance veterinarian listing workflow demonstrating caching strategies and multi-format API responses.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant VC as VetController
    participant VR as VetRepository
    participant Cache as JCache/Caffeine
    participant DB as Database

    Note over U,DB: HTML View Request
    U->>VC: GET /vets.html?page=1
    VC->>VR: findAll(PageRequest)
    
    alt Cache Hit
        VR->>Cache: get("vets::findAll")
        Cache-->>VR: Cached vet collection
    else Cache Miss
        VR->>DB: SELECT vets with specialties (EAGER fetch)
        DB-->>VR: Complete vet data
        VR->>Cache: put("vets::findAll", vetData)
    end
    
    VR-->>VC: Page<Vet> with specialties
    VC-->>U: Render vet list HTML

    Note over U,DB: JSON API Request
    U->>VC: GET /vets (Accept: application/json)
    VC->>VR: findAll()
    
    alt Cache Hit
        VR->>Cache: get("vets::findAll")
        Cache-->>VR: Cached vet collection
    else Cache Miss
        VR->>DB: SELECT vets with specialties
        DB-->>VR: Vet data
        VR->>Cache: put("vets::findAll", vetData)
    end
    
    VR-->>VC: Vet collection
    VC-->>U: JSON response with vet data

    Note over U,DB: Cache Invalidation
    Note over VR,Cache: When vet data changes
    VR->>Cache: evict("vets::findAll")
    Cache-->>VR: Cache cleared
```

### Communication Patterns
- **Caching Strategy**: JCache with Caffeine backend for performance
- **Multi-format Responses**: Content negotiation (HTML/JSON/XML)
- **Eager Loading**: Vet-specialty relationship loading
- **Cache Invalidation**: Manual eviction on data changes

---

## Workflow 4: Error Handling and System Resilience

### Description
Comprehensive error handling workflow showing validation failures, exception scenarios, and recovery patterns.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant PC as PetController
    participant CC as CrashController
    participant PV as PetValidator
    participant OR as OwnerRepository
    participant DB as Database
    participant EL as ErrorLogger

    Note over U,EL: Validation Error Scenario
    U->>OC: POST /owners/new (invalid telephone)
    OC->>PV: Validate owner data
    PV-->>OC: Validation failed - telephone pattern
    OC-->>U: Render form with error messages

    Note over U,EL: Business Rule Violation
    U->>PC: POST /pets/new (duplicate pet name)
    PC->>PV: Validate pet data
    PV->>OR: Check existing pets for owner
    OR->>DB: SELECT pets by owner
    DB-->>OR: Existing pets
    OR-->>PV: Pet list
    PV-->>PC: Validation failed - duplicate name
    PC-->>U: Render form with business error

    Note over U,EL: Database Error Scenario
    U->>OC: GET /owners/{id}
    OC->>OR: findById(invalidId)
    OR->>DB: SELECT owner by ID
    DB-->>OR: EmptyResultDataAccessException
    OR-->>OC: Exception thrown
    OC->>EL: Log database error
    OC-->>U: Redirect to error page with message

    Note over U,EL: System Exception Testing
    U->>CC: GET /oups
    CC->>CC: throw new RuntimeException("Expected exception")
    CC->>EL: Log exception with stack trace
    CC-->>U: Custom error page with details

    Note over U,EL: Recovery Pattern
    U->>OC: GET /owners/find (after error)
    OC-->>U: Normal search form (system recovered)
```

### Communication Patterns
- **Declarative Validation**: Bean Validation with custom constraints
- **Exception Handling**: Spring MVC @ExceptionHandler for graceful degradation
- **Error Logging**: Structured logging for operational visibility
- **User Feedback**: Meaningful error messages with recovery guidance

---

## Workflow 5: Internationalization and Configuration

### Description
Dynamic locale resolution and configuration management workflow supporting multi-language deployment.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant WC as WelcomeController
    participant WConf as WebConfiguration
    participant MC as MessageSource
    participant TC as Thymeleaf
    participant LC as LocaleResolver

    Note over U,LC: Initial Page Load
    U->>WC: GET / (no locale)
    WC->>LC: resolveLocale(request)
    LC-->>WC: Default locale (en)
    WC->>MC: getMessage("welcome", [], locale)
    MC-->>WC: "Welcome" message
    WC->>TC: render with locale context
    TC-->>WC: Localized HTML
    WC-->>U: Welcome page in English

    Note over U,LC: Language Change Request
    U->>WC: GET /?lang=es
    WConf->>LC: setLocale(request, response, "es")
    LC-->>WConf: Locale set to Spanish
    WC->>MC: getMessage("welcome", [], es_LOCALE)
    MC-->>WC: "Bienvenido" message
    WC->>TC: render with Spanish context
    TC-->>WC: Localized HTML in Spanish
    WC-->>U: Welcome page in Spanish

    Note over U,LC: Session-based Locale Persistence
    U->>WC: GET /owners (subsequent request)
    WC->>LC: resolveLocale(request)
    LC-->>WC: Spanish locale from session
    WC->>MC: getMessage("search", [], es_LOCALE)
    MC-->>WC: "Buscar" message
    WC->>TC: render with persisted locale
    TC-->>WC: Consistent Spanish UI
    WC-->>U: Owners page in Spanish
```

### Communication Patterns
- **Session-based Localization**: Persistent locale storage across requests
- **Message Source Abstraction**: Externalized message properties
- **Content Negotiation**: Dynamic language switching via URL parameters
- **Template Resolution**: Thymeleaf integration with locale context

---

## Workflow 6: Data Access Patterns and Repository Interactions

### Description
Comprehensive data access workflow showing repository patterns, relationship loading, and transaction management.

### Sequence Diagram

```mermaid
sequenceDiagram
    participant C as Controller
    participant OR as OwnerRepository
    participant PR as PetRepository
    participant VR as VisitRepository
    participant VTR as VetRepository
    participant PT as PetTypeRepository
    participant EM as EntityManager
    participant TX as TransactionManager
    participant DB as Database

    Note over C,DB: Complex Query Execution
    C->>OR: findByLastNameStartingWith("Smith", PageRequest)
    OR->>EM: createQuery("SELECT o FROM Owner o...")
    EM->>DB: Execute paginated SQL query
    DB-->>EM: Result set
    EM-->>OR: Page<Owner> with metadata
    OR-->>C: Paginated results

    Note over C,DB: Relationship Loading
    C->>OR: findById(ownerId)
    OR->>EM: find(Owner.class, ownerId)
    EM->>DB: SELECT owner data
    DB-->>EM: Owner record
    EM->>DB: SELECT pets WHERE owner_id = ? (EAGER fetch)
    DB-->>EM: Pet records
    EM->>DB: SELECT visits WHERE pet_id IN (?) (EAGER fetch)
    DB-->>EM: Visit records
    EM-->>OR: Complete owner graph
    OR-->>C: Owner with pets and visits

    Note over C,DB: Cached Repository Access
    C->>VTR: findAll()
    VTR->>TX: Begin transaction
    VTR->>EM: createQuery("SELECT v FROM Vet v...")
    EM->>DB: SELECT vets with specialties
    DB-->>EM: Vet data
    EM-->>VTR: Vet collection
    VTR->>TX: Commit transaction
    VTR->>Cache: put("vets::findAll", result)
    VTR-->>C: Cached vet data

    Note over C,DB: Transactional Update
    C->>PR: save(pet)
    PR->>TX: Begin transaction
    PR->>EM: merge(pet)
    EM->>DB: UPDATE pets SET ...
    DB-->>EM: Update confirmation
    EM-->>PR: Managed entity
    PR->>TX: Commit transaction
    PR-->>C: Saved pet entity

    Note over C,DB: Reference Data Access
    C->>PT: findAll()
    PT->>EM: createQuery("SELECT pt FROM PetType pt")
    EM->>DB: SELECT pet_types
    DB-->>EM: Reference data
    EM-->>PT: Pet type list
    PT-->>C: Static reference data
```

### Communication Patterns
- **Repository Abstraction**: Spring Data JPA repository pattern
- **Eager/Lazy Loading**: Configurable relationship fetching strategies
- **Transaction Management**: JPA transaction boundaries
- **Cached Access**: Performance optimization for read-heavy operations
- **Pagination**: Database-level result set pagination
```