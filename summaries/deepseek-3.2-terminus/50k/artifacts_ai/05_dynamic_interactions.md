```markdown
# Spring PetClinic Dynamic Interaction Flows

## Workflow 1: Owner Registration and Pet Management

### Description
This workflow covers the complete lifecycle of registering a new pet owner and managing their pets. It starts with owner registration, followed by pet registration, and potentially visit scheduling.

### Communication Patterns
- **Synchronous**: REST form submissions, database transactions
- **Database**: JPA/Hibernate entity persistence
- **Validation**: Bean validation with custom validators

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Client Browser
    participant OwnerCtrl as OwnerController
    participant PetCtrl as PetController
    participant VisitCtrl as VisitController
    participant OwnerRepo as OwnerRepository
    participant PetRepo as PetRepository
    participant VisitRepo as VisitRepository
    participant DB as Database

    Note over User,DB: Owner Registration Phase
    User->>OwnerCtrl: GET /owners/new
    OwnerCtrl-->>User: Return owner creation form
    
    User->>OwnerCtrl: POST /owners/new (owner data)
    OwnerCtrl->>OwnerCtrl: Validate owner data
    OwnerCtrl->>OwnerRepo: save(owner)
    OwnerRepo->>DB: INSERT INTO owners
    DB-->>OwnerRepo: Owner record created
    OwnerRepo-->>OwnerCtrl: Saved owner entity
    OwnerCtrl-->>User: Redirect to owner details
    
    Note over User,DB: Pet Registration Phase
    User->>PetCtrl: GET /owners/{id}/pets/new
    PetCtrl->>PetRepo: findPetTypes()
    PetRepo->>DB: SELECT * FROM types
    DB-->>PetRepo: Pet types
    PetCtrl-->>User: Return pet creation form with types
    
    User->>PetCtrl: POST /owners/{id}/pets/new (pet data)
    PetCtrl->>PetCtrl: Validate pet (PetValidator)
    PetCtrl->>PetRepo: save(pet)
    PetRepo->>DB: INSERT INTO pets
    DB-->>PetRepo: Pet record created
    PetRepo-->>PetCtrl: Saved pet entity
    PetCtrl-->>User: Redirect to owner details
    
    Note over User,DB: Visit Scheduling Phase
    User->>VisitCtrl: GET /owners/{id}/pets/{petId}/visits/new
    VisitCtrl-->>User: Return visit creation form
    
    User->>VisitCtrl: POST /owners/{id}/pets/{petId}/visits/new (visit data)
    VisitCtrl->>VisitCtrl: Validate visit description
    VisitCtrl->>VisitRepo: save(visit)
    VisitRepo->>DB: INSERT INTO visits
    DB-->>VisitRepo: Visit record created
    VisitRepo-->>VisitCtrl: Saved visit entity
    VisitCtrl-->>User: Redirect to owner details
```

## Workflow 2: Owner Search and Profile Management

### Description
This workflow handles searching for existing owners, viewing their profiles, and updating owner information. It demonstrates pagination and search functionality.

### Communication Patterns
- **Synchronous**: REST GET/POST requests
- **Database**: JPA query methods with pagination
- **Caching**: None (owner data not cached)

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Client Browser
    participant OwnerCtrl as OwnerController
    participant OwnerRepo as OwnerRepository
    participant DB as Database

    Note over User,DB: Owner Search Phase
    User->>OwnerCtrl: GET /owners/find
    OwnerCtrl-->>User: Return search form
    
    User->>OwnerCtrl: GET /owners?lastName=Smith&page=1
    OwnerCtrl->>OwnerRepo: findByLastNameContaining("Smith", PageRequest)
    OwnerRepo->>DB: SELECT * FROM owners WHERE last_name LIKE '%Smith%' LIMIT 5
    DB-->>OwnerRepo: Matching owners (max 5)
    OwnerRepo-->>OwnerCtrl: Page<Owner> result
    OwnerCtrl-->>User: Return search results with pagination
    
    Note over User,DB: Owner Profile View
    User->>OwnerCtrl: GET /owners/{id}
    OwnerCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT * FROM owners WHERE id = ?
    DB-->>OwnerRepo: Owner record
    OwnerRepo->>DB: SELECT * FROM pets WHERE owner_id = ? (Eager fetch)
    DB-->>OwnerRepo: Pet records
    OwnerRepo->>DB: SELECT * FROM visits WHERE pet_id IN (...) (Eager fetch)
    DB-->>OwnerRepo: Visit records
    OwnerRepo-->>OwnerCtrl: Owner with pets and visits
    OwnerCtrl-->>User: Return owner profile page
    
    Note over User,DB: Owner Update
    User->>OwnerCtrl: GET /owners/{id}/edit
    OwnerCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT * FROM owners WHERE id = ?
    DB-->>OwnerRepo: Owner record
    OwnerCtrl-->>User: Return edit form with current data
    
    User->>OwnerCtrl: POST /owners/{id}/edit (updated data)
    OwnerCtrl->>OwnerCtrl: Validate updated data
    OwnerCtrl->>OwnerRepo: save(owner)
    OwnerRepo->>DB: UPDATE owners SET ...
    DB-->>OwnerRepo: Record updated
    OwnerCtrl-->>User: Redirect to owner details
```

## Workflow 3: Veterinarian Directory Access

### Description
This workflow covers accessing the veterinarian directory, which demonstrates caching and API content negotiation patterns.

### Communication Patterns
- **Synchronous**: REST requests with content negotiation
- **Caching**: JCache with Caffeine provider
- **Database**: Repository with @Cacheable annotation

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Client Browser
    participant API as API Client
    participant VetCtrl as VetController
    participant VetRepo as VetRepository
    participant Cache as Caffeine Cache
    participant DB as Database

    Note over User,DB: HTML View Request
    User->>VetCtrl: GET /vets.html
    VetCtrl->>VetRepo: findAll()
    
    alt Cache Hit
        VetRepo->>Cache: Get "vets" cache entry
        Cache-->>VetRepo: Cached vet list
    else Cache Miss
        VetRepo->>DB: SELECT v.*, s.* FROM vets v <br/>LEFT JOIN vet_specialties vs ON v.id = vs.vet_id <br/>LEFT JOIN specialties s ON vs.specialty_id = s.id
        DB-->>VetRepo: Vet records with specialties
        VetRepo->>Cache: Put "vets" cache entry
    end
    
    VetRepo-->>VetCtrl: List<Vet> with specialties
    VetCtrl-->>User: Return HTML view with vet list

    Note over API,DB: JSON API Request
    API->>VetCtrl: GET /vets (Accept: application/json)
    VetCtrl->>VetRepo: findAll()
    
    alt Cache Hit
        VetRepo->>Cache: Get "vets" cache entry
        Cache-->>VetRepo: Cached vet list
    else Cache Miss
        VetRepo->>DB: SELECT with joins
        DB-->>VetRepo: Vet records with specialties
        VetRepo->>Cache: Put "vets" cache entry
    end
    
    VetRepo-->>VetCtrl: List<Vet> with specialties
    VetCtrl-->>API: Return JSON response
```

## Workflow 4: Error Handling and Validation

### Description
This workflow demonstrates the system's error handling and validation patterns, including custom validation logic and error page rendering.

### Communication Patterns
- **Synchronous**: Form validation and error responses
- **Validation**: Bean Validation with custom validators
- **Error Handling**: Spring MVC error resolution

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Client Browser
    participant OwnerCtrl as OwnerController
    participant PetCtrl as PetController
    participant PetVal as PetValidator
    participant View as Thymeleaf Templates
    participant ErrCtrl as ErrorController

    Note over User,ErrCtrl: Validation Error Scenario
    User->>OwnerCtrl: POST /owners/new (invalid data)
    OwnerCtrl->>OwnerCtrl: @Valid binding result
    OwnerCtrl->>OwnerCtrl: hasErrors() = true
    OwnerCtrl-->>User: Return form with field errors
    
    Note over User,ErrCtrl: Custom Validation Error
    User->>PetCtrl: POST /owners/{id}/pets/new (duplicate pet name)
    PetCtrl->>PetVal: validate(pet, errors)
    PetVal->>PetVal: Check name uniqueness per owner
    PetVal-->>PetCtrl: Validation error added
    PetCtrl-->>User: Return form with custom error
    
    Note over User,ErrCtrl: System Error Scenario
    User->>ErrCtrl: GET /oups (error simulation)
    ErrCtrl->>ErrCtrl: throw new Exception("Expected exception")
    
    ErrCtrl->>View: error.html template
    View-->>User: Custom error page (non-whitelabel)
    
    Note over User,ErrCtrl: Runtime Exception
    User->>OwnerCtrl: GET /owners/999 (non-existent)
    OwnerCtrl->>OwnerCtrl: findById(999) returns empty
    OwnerCtrl->>View: Error handling
    View-->>User: Appropriate error message
```

## Workflow 5: Internationalization and Localization

### Description
This workflow shows how the application handles internationalization, including locale resolution and message bundle management.

### Communication Patterns
- **Synchronous**: HTTP request with locale context
- **Localization**: Spring MessageSource with property files
- **Templating**: Thymeleaf with i18n expressions

### Sequence Diagram

```mermaid
sequenceDiagram
    participant User as Client Browser
    participant WebConfig as WebConfiguration
    participant LocaleRes as LocaleResolver
    participant MessageSrc as MessageSource
    participant View as Thymeleaf Templates
    participant OwnerCtrl as OwnerController

    User->>OwnerCtrl: GET /owners/find (Accept-Language: fr)
    OwnerCtrl->>LocaleRes: resolveLocale(request)
    LocaleRes-->>OwnerCtrl: Locale.FRENCH
    
    OwnerCtrl->>View: owners/findOwners.html
    View->>MessageSrc: getMessage("welcome", null, locale)
    MessageSrc-->>View: "Bienvenue" from messages_fr.properties
    View->>MessageSrc: getMessage("find.owners", null, locale)
    MessageSrc-->>View: "Trouver des propriétaires"
    View->>MessageSrc: getMessage("error.required", null, locale)
    MessageSrc-->>View: "Ce champ est obligatoire"
    
    View-->>User: Rendered French HTML page
    
    Note over User,View: Validation Messages
    User->>OwnerCtrl: POST /owners/new (invalid - French locale)
    OwnerCtrl->>OwnerCtrl: @Valid with binding result
    OwnerCtrl->>MessageSrc: getMessage(bindingError, locale)
    MessageSrc-->>OwnerCtrl: French error message
    OwnerCtrl-->>User: Form with French validation errors
```

## Workflow 6: Database Integration and Transaction Management

### Description
This workflow demonstrates the database interaction patterns, including transaction management and repository operations across multiple entities.

### Communication Patterns
- **Synchronous**: JPA/Hibernate entity operations
- **Transactions**: Spring @Transactional management
- **Database**: Connection pooling and ORM mapping

### Sequence Diagram

```mermaid
sequenceDiagram
    participant Service as ClinicService
    participant OwnerRepo as OwnerRepository
    participant PetRepo as PetRepository
    participant VisitRepo as VisitRepository
    participant TX as Transaction Manager
    participant EM as EntityManager
    participant DB as Database

    Note over Service,DB: Transactional Owner-Pet-Visit Creation
    Service->>TX: Begin Transaction
    
    Service->>OwnerRepo: save(owner)
    OwnerRepo->>EM: persist(owner)
    EM->>DB: INSERT INTO owners (transactional)
    DB-->>EM: Owner ID generated
    EM-->>OwnerRepo: Managed owner entity
    
    Service->>PetRepo: save(pet)
    PetRepo->>EM: persist(pet)
    EM->>DB: INSERT INTO pets (transactional)
    DB-->>EM: Pet ID generated
    EM-->>PetRepo: Managed pet entity
    
    Service->>VisitRepo: save(visit)
    VisitRepo->>EM: persist(visit)
    EM->>DB: INSERT INTO visits (transactional)
    DB-->>EM: Visit ID generated
    EM-->>VisitRepo: Managed visit entity
    
    Service->>TX: Commit Transaction
    TX->>EM: flush and commit
    EM->>DB: COMMIT
    DB-->>EM: Commit successful
    EM-->>TX: Transaction committed
    
    Note over Service,DB: Read Operation with Relationships
    Service->>OwnerRepo: findByIdWithPets(ownerId)
    OwnerRepo->>EM: find(Owner.class, ownerId)
    EM->>DB: SELECT * FROM owners WHERE id = ?
    DB-->>EM: Owner record
    EM->>DB: SELECT * FROM pets WHERE owner_id = ? (Lazy/Eager)
    DB-->>EM: Pet records
    EM-->>OwnerRepo: Owner with pet collection
```

## Workflow 7: Caching Strategy Implementation

### Description
This workflow illustrates the caching mechanism used for veterinarian data, demonstrating cache-aside pattern with JCache and Caffeine.

### Communication Patterns
- **Caching**: JCache API with Caffeine provider
- **Pattern**: Cache-Aside (Lazy Loading)
- **Monitoring**: JMX cache statistics

### Sequence Diagram

```mermaid
sequenceDiagram
    participant VetCtrl as VetController
    participant VetRepo as VetRepository
    participant Cache as Caffeine Cache
    participant CacheConfig as CacheConfiguration
    participant DB as Database
    participant JMX as JMX Monitor

    Note over VetCtrl,JMX: Cache Initialization
    CacheConfig->>CacheConfig: Create Caffeine cache
    CacheConfig->>Cache: Configure with size/time settings
    CacheConfig->>JMX: Enable cache statistics
    
    Note over VetCtrl,JMX: Cache Miss Scenario
    VetCtrl->>VetRepo: findAll()
    VetRepo->>Cache: Get "vets" key
    Cache-->>VetRepo: Cache miss
    
    VetRepo->>DB: SELECT vets with specialties join
    DB-->>VetRepo: Vet data
    VetRepo->>Cache: Put "vets" key with data
    Cache->>JMX: Record cache miss statistic
    VetRepo-->>VetCtrl: Vet list
    
    Note over VetCtrl,JMX: Cache Hit Scenario
    VetCtrl->>VetRepo: findAll()
    VetRepo->>Cache: Get "vets" key
    Cache-->>VetRepo: Cached vet data
    Cache->>JMX: Record cache hit statistic
    VetRepo-->>VetCtrl: Vet list (from cache)
    
    Note over VetCtrl,JMX: Cache Eviction
    VetCtrl->>VetRepo: save(vet) - modification
    VetRepo->>Cache: Evict "vets" key
    Cache->>JMX: Record eviction statistic
    VetRepo->>DB: UPDATE vets...
```