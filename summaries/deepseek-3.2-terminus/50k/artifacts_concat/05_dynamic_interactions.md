```markdown
# Spring PetClinic Dynamic Interaction Flows

## Workflow 1: Owner Registration and Pet Management

### Description
This workflow captures the complete process of registering a new owner and managing their pets. It's triggered when a new client visits the veterinary clinic and needs to be added to the system along with their pets.

### Communication Patterns
- Synchronous REST calls for form submissions
- Database transactions for data persistence
- Server-side validation with custom validators
- Form-based navigation with redirects

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant PC as PetController
    participant OV as OwnerValidator
    participant PV as PetValidator
    participant OR as OwnerRepository
    participant PR as PetRepository
    participant PTR as PetTypeRepository
    participant DB as Database

    Note over U,DB: Owner Registration Phase
    U->>OC: GET /owners/new
    OC->>U: Return owner creation form
    
    U->>OC: POST /owners/new (owner data)
    OC->>OV: Validate owner data
    OV->>OC: Validation result
    alt Validation Failed
        OC->>U: Return form with errors
    else Validation Passed
        OC->>OR: save(owner)
        OR->>DB: INSERT INTO owners
        DB->>OR: Owner ID
        OR->>OC: Saved owner
        OC->>U: Redirect to /owners/{id}
    end

    Note over U,DB: Pet Registration Phase
    U->>PC: GET /owners/{id}/pets/new
    PC->>PTR: findAllPetTypes()
    PTR->>DB: SELECT * FROM types
    DB->>PTR: Pet types
    PTR->>PC: Pet type list
    PC->>U: Return pet creation form
    
    U->>PC: POST /owners/{id}/pets/new (pet data)
    PC->>PV: Validate pet data
    PV->>PR: Check pet name uniqueness
    PR->>DB: SELECT pets by owner & name
    DB->>PR: Existing pets
    PR->>PV: Validation result
    PV->>PC: Validation result
    
    alt Validation Failed
        PC->>U: Return form with errors
    else Validation Passed
        PC->>PR: save(pet)
        PR->>DB: INSERT INTO pets
        DB->>PR: Pet ID
        PR->>PC: Saved pet
        PC->>U: Redirect to /owners/{id}
    end
```

## Workflow 2: Veterinary Visit Scheduling

### Description
This workflow handles the scheduling and recording of veterinary visits for pets. It's triggered when a pet owner brings their animal for a medical checkup or treatment.

### Communication Patterns
- Form-based data submission
- Automatic date handling (current date)
- Database transactions with entity relationships
- Redirect-after-POST pattern

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant VC as VisitController
    participant OR as OwnerRepository
    participant PR as PetRepository
    participant VR as VisitRepository
    participant DB as Database

    U->>VC: GET /owners/{ownerId}/pets/{petId}/visits/new
    VC->>OR: findById(ownerId)
    OR->>DB: SELECT * FROM owners WHERE id=?
    DB->>OR: Owner data
    OR->>VC: Owner entity
    
    VC->>PR: findById(petId)
    PR->>DB: SELECT * FROM pets WHERE id=?
    DB->>PR: Pet data
    PR->>VC: Pet entity
    
    VC->>U: Return visit creation form
    
    U->>VC: POST /owners/{ownerId}/pets/{petId}/visits/new (visit data)
    VC->>VC: Set visit date to current date
    VC->>VR: save(visit)
    VR->>DB: INSERT INTO visits (date, description, pet_id)
    DB->>VR: Visit ID
    VR->>VC: Saved visit
    VC->>U: Redirect to /owners/{ownerId}
```

## Workflow 3: Owner Search and Pagination

### Description
This workflow handles the search functionality for finding owners by last name with pagination support. It's triggered when clinic staff needs to locate a client's record.

### Communication Patterns
- Search form submission
- Database queries with LIKE conditions
- Pagination handling (5 records per page)
- Server-side rendering with Thymeleaf

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant OR as OwnerRepository
    participant DB as Database

    U->>OC: GET /owners/find
    OC->>U: Return search form
    
    U->>OC: GET /owners?lastName=Smith&page=1
    alt Empty Search
        OC->>OR: findAll(pageable)
    else With Search Term
        OC->>OR: findByLastNameContaining(lastName, pageable)
    end
    
    OR->>DB: SELECT with LIMIT 5 OFFSET calculation
    DB->>OR: Owner records
    OR->>OC: Page<Owner> result
    
    OC->>U: Render search results with pagination links
```

## Workflow 4: Veterinarian Information Retrieval

### Description
This workflow handles the retrieval of veterinarian information with caching support. It's triggered when displaying the clinic staff information on the website.

### Communication Patterns
- Cached data access with JCache/Caffeine
- Multiple response formats (HTML/JSON/XML)
- Asynchronous cache population
- JMX monitoring integration

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant VC as VetController
    participant VR as VetRepository
    participant Cache as JCache Manager
    participant DB as Database

    U->>VC: GET /vets.html
    VC->>VR: findAll()
    
    VR->>Cache: Check cache for "vets"
    alt Cache Hit
        Cache->>VR: Cached vet list
        VR->>VC: Return cached data
    else Cache Miss
        VR->>DB: SELECT vets with specialties
        DB->>VR: Vet data with joins
        VR->>Cache: Store in cache "vets"
        VR->>VC: Return database data
    end
    
    VC->>U: Render HTML vet list
    
    Note right of Cache: Cache statistics tracked for JMX
    
    U->>VC: GET /vets (Accept: application/json)
    VC->>VR: findAll()
    VR->>Cache: Check cache
    Cache->>VR: Cached data
    VR->>VC: Vet list
    VC->>U: Return JSON response
```

## Workflow 5: Error Handling and Validation

### Description
This workflow demonstrates the comprehensive error handling and validation patterns throughout the application, including custom validators and internationalized error messages.

### Communication Patterns
- Bean validation with custom validators
- Internationalized error messages
- Form re-rendering with error display
- Custom error page handling

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant OC as OwnerController
    participant PC as PetController
    participant OV as OwnerValidator
    participant PV as PetValidator
    participant OR as OwnerRepository
    participant DB as Database
    participant TM as Thymeleaf Templates
    participant IL as I18N MessageSource

    Note over U,IL: Owner Validation Failure
    U->>OC: POST /owners/new (invalid data)
    OC->>OV: validate(owner, errors)
    OV->>OV: Check telephone format (10 digits)
    OV->>OV: Check required fields
    OV->>IL: getMessage(validation.error.telephone)
    IL->>OV: Localized error message
    OV->>OC: Validation errors
    
    OC->>TM: render with bindingResult
    TM->>U: Form with field errors highlighted
    
    Note over U,IL: Pet Validation Failure
    U->>PC: POST /owners/{id}/pets/new (invalid pet)
    PC->>PV: validate(pet, errors)
    PV->>OR: Check pet name uniqueness
    OR->>DB: SELECT pets by owner & name
    DB->>OR: Existing pets
    OR->>PV: Duplicate check result
    PV->>PV: Check birth date not in future
    PV->>IL: getMessage(validation.error.duplicate)
    IL->>PV: Localized error message
    PV->>PC: Validation errors
    
    PC->>TM: render with bindingResult
    TM->>U: Form with pet validation errors
    
    Note over U,IL: System Error Handling
    U->>OC: GET /oups (error simulation)
    OC->>OC: Throw runtime exception
    OC->>TM: Render custom error page
    TM->>U: Non-whitelabel error page
```

## Workflow 6: Multi-Database Deployment and Testing

### Description
This workflow shows the database interaction patterns across different deployment scenarios, including profile-based database selection and Testcontainers-based integration testing.

### Communication Patterns
- Profile-based data source configuration
- Testcontainers for isolated database testing
- JPA repository abstraction
- Transaction management

```mermaid
sequenceDiagram
    participant App as Spring Application
    participant DS as DataSource
    participant JPA as JPA EntityManager
    participant OR as OwnerRepository
    participant DB as Active Database
    participant TC as TestContainers

    Note over App,DB: Production Deployment
    App->>App: Load spring.profiles.active=mysql
    App->>DS: Configure MySQL DataSource
    DS->>DB: Connect to jdbc:mysql://localhost/petclinic
    
    App->>OR: save(owner)
    OR->>JPA: persist(owner)
    JPA->>DB: INSERT INTO owners
    DB->>JPA: Generated ID
    JPA->>OR: Managed entity
    OR->>App: Saved owner
    
    Note over App,DB: Integration Testing
    App->>TC: Start MySQL container
    TC->>TC: Create temporary database
    TC->>DB: Expose container port
    
    App->>DS: Connect to Testcontainers MySQL
    DS->>DB: JDBC connection
    App->>OR: findAll()
    OR->>JPA: createQuery("SELECT o FROM Owner o")
    JPA->>DB: SELECT * FROM owners
    DB->>JPA: Result set
    JPA->>OR: List<Owner>
    OR->>App: Owner list
    
    App->>TC: Stop and remove container
```

## Workflow 7: Performance Testing and Monitoring

### Description
This workflow demonstrates the performance testing patterns and monitoring capabilities, including JMeter load testing and Actuator endpoints for system observability.

### Communication Patterns
- JMeter-based load testing
- Spring Boot Actuator monitoring
- Database connection pooling
- Caffeine cache statistics

```mermaid
sequenceDiagram
    participant JM as JMeter
    participant App as Application
    participant OC as OwnerController
    participant VC as VetController
    participant OR as OwnerRepository
    participant VR as VetRepository
    participant Cache as Cache
    participant DB as Database
    participant Actuator as Spring Actuator

    JM->>App: Simulate 500 concurrent users
    
    par Owner Search Workload
        JM->>OC: GET /owners/find
        OC->>U: Search form
        JM->>OC: GET /owners?lastName=Smith
        OC->>OR: findByLastNameContaining()
        OR->>DB: SELECT with LIKE condition
        DB->>OR: Results
        OR->>OC: Owner list
        OC->>JM: Search results
    and Vet Listing Workload
        JM->>VC: GET /vets
        VC->>VR: findAll()
        VR->>Cache: Get cached vets
        Cache->>VR: Cached data
        VR->>VC: Vet list
        VC->>JM: JSON response
    and Visit Creation Workload
        JM->>VC: POST /visits/new
        VC->>DB: INSERT visit record
        DB->>VC: Success
        VC->>JM: Redirect response
    end
    
    Note over JM,Actuator: Monitoring Phase
    JM->>Actuator: GET /actuator/metrics
    Actuator->>App: Collect JVM, cache metrics
    App->>Actuator: System statistics
    Actuator->>JM: Performance metrics
    
    JM->>Actuator: GET /actuator/caches
    Actuator->>Cache: Get cache statistics
    Cache->>Actuator: Cache hit/miss ratios
    Actuator->>JM: Cache performance data
```