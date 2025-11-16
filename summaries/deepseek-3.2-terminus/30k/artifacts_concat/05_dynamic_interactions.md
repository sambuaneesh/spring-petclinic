```markdown
## Dynamic Interaction Flows for Spring PetClinic

### Workflow 1: Owner Registration and Pet Management

**Purpose**: Complete workflow for registering a new owner and adding pets to their account
**Triggers**: User accesses owner registration form via `/owners/new`

```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant Controller as OwnerController
    participant Service as ClinicService
    participant Repo as OwnerRepository
    participant DB as Database
    participant Cache as Cache

    User->>Browser: Navigate to /owners/new
    Browser->>Controller: GET /owners/new
    Controller->>Browser: Return owner creation form
    
    User->>Browser: Fill owner details & submit
    Browser->>Controller: POST /owners/new (owner data)
    Controller->>Controller: Validate owner fields
    Controller->>Service: saveOwner(owner)
    Service->>Repo: save(owner)
    Repo->>DB: INSERT INTO owners
    DB->>Repo: Return saved owner with ID
    Repo->>Service: Return owner entity
    Service->>Controller: Return saved owner
    Controller->>Browser: Redirect to /owners/{ownerId}
    
    Browser->>Controller: GET /owners/{ownerId}
    Controller->>Service: findOwnerById(ownerId)
    Service->>Repo: findById(ownerId)
    Repo->>DB: SELECT * FROM owners WHERE id=?
    DB->>Repo: Return owner data
    Repo->>Service: Return owner entity
    Service->>Controller: Return owner with empty pets
    Controller->>Browser: Display owner details page
    
    User->>Browser: Click "Add New Pet"
    Browser->>Controller: GET /owners/{ownerId}/pets/new
    Controller->>Service: findPetTypes()
    Service->>Repo: PetTypeRepository.findAll()
    Repo->>DB: SELECT * FROM types
    DB->>Repo: Return pet types
    Repo->>Service: Return pet types
    Service->>Controller: Return pet types
    Controller->>Browser: Return pet creation form
    
    User->>Browser: Fill pet details & submit
    Browser->>Controller: POST /owners/{ownerId}/pets/new (pet data)
    Controller->>Controller: Validate pet (PetValidator)
    Controller->>Service: savePet(ownerId, pet)
    Service->>Service: Verify pet uniqueness per owner
    Service->>Repo: save(pet)
    Repo->>DB: INSERT INTO pets
    DB->>Repo: Return saved pet with ID
    Repo->>Service: Return pet entity
    Service->>Controller: Return saved pet
    Controller->>Browser: Redirect to /owners/{ownerId}
```

**Communication Patterns**: 
- Synchronous REST calls for form submissions
- Database transactions for data persistence
- Server-side validation with custom validators
- Redirect-after-POST pattern to prevent duplicate submissions

---

### Workflow 2: Veterinary Visit Scheduling

**Purpose**: Schedule and record veterinary visits for pets
**Triggers**: User navigates to visit creation form for specific pet

```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant VisitCtrl as VisitController
    participant OwnerCtrl as OwnerController
    participant Service as ClinicService
    participant OwnerRepo as OwnerRepository
    participant VisitRepo as VisitRepository
    participant DB as Database

    User->>Browser: Navigate to owner details
    Browser->>OwnerCtrl: GET /owners/{ownerId}
    OwnerCtrl->>Service: findOwnerById(ownerId)
    Service->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owners + JOIN pets + JOIN visits
    DB->>OwnerRepo: Return owner with pets and visits
    OwnerRepo->>Service: Return owner entity
    Service->>OwnerCtrl: Return owner with populated data
    OwnerCtrl->>Browser: Display owner with pets and visit history
    
    User->>Browser: Click "Add Visit" on pet
    Browser->>VisitCtrl: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitCtrl->>Service: findOwnerById(ownerId)
    Service->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owner with specified pet
    DB->>OwnerRepo: Return owner and pet data
    OwnerRepo->>Service: Return owner entity
    Service->>VisitCtrl: Return owner with specified pet
    VisitCtrl->>Browser: Return visit creation form (pre-filled pet info)
    
    User->>Browser: Fill visit details & submit
    Browser->>VisitCtrl: POST /owners/{ownerId}/pets/{petId}/visits/new
    VisitCtrl->>VisitCtrl: Validate visit (description required)
    VisitCtrl->>Service: saveVisit(visit)
    Service->>VisitRepo: save(visit)
    VisitRepo->>DB: INSERT INTO visits (date, description, pet_id)
    DB->>VisitRepo: Return saved visit with ID
    VisitRepo->>Service: Return visit entity
    Service->>VisitCtrl: Return saved visit
    VisitCtrl->>Browser: Redirect to /owners/{ownerId}
    
    Note over Browser,DB: Visit now appears in pet's visit history
```

**Communication Patterns**:
- Form-based synchronous HTTP requests
- Eager fetching of related entities (Owner → Pets → Visits)
- Transactional database operations
- Automatic date population (defaults to current date)

---

### Workflow 3: Veterinarian Directory Access

**Purpose**: Display paginated list of veterinarians with their specialties
**Triggers**: User navigates to veterinarian listing page

```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant Controller as VetController
    participant Service as ClinicService
    participant Repo as VetRepository
    participant Cache as Cache Layer
    participant DB as Database

    User->>Browser: Navigate to /vets.html
    Browser->>Controller: GET /vets.html?page=1
    Controller->>Cache: Check cache for "vets"
    alt Cache Hit
        Cache->>Controller: Return cached vet list
    else Cache Miss
        Controller->>Service: findVets(pageable)
        Service->>Repo: findAll(pageable)
        Repo->>DB: SELECT vets + JOIN specialties (LIMIT 5 OFFSET 0)
        DB->>Repo: Return vets with specialties
        Repo->>Service: Return paginated vets
        Service->>Controller: Return vet data
        Controller->>Cache: Store vet list in cache
    end
    Controller->>Browser: Render vet list template
    
    User->>Browser: Scroll to bottom, click next page
    Browser->>Controller: GET /vets.html?page=2
    Controller->>Service: findVets(pageable)
    Service->>Repo: findAll(pageable)
    Repo->>DB: SELECT vets + JOIN specialties (LIMIT 5 OFFSET 5)
    DB->>Repo: Return next page of vets
    Repo->>Service: Return paginated vets
    Service->>Controller: Return vet data
    Controller->>Browser: Render vet list template page 2
    
    Note over User,DB: API endpoints also available for JSON/XML responses
```

**Communication Patterns**:
- Cache-aside pattern for performance optimization
- Database pagination with Spring Data Pageable
- Eager fetching of vet specialties
- Multi-format response support (HTML/JSON/XML)

---

### Workflow 4: Owner Search and Discovery

**Purpose**: Search for owners by last name with paginated results
**Triggers**: User accesses owner search functionality

```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant Controller as OwnerController
    participant Service as ClinicService
    participant Repo as OwnerRepository
    participant DB as Database

    User->>Browser: Navigate to /owners/find
    Browser->>Controller: GET /owners/find
    Controller->>Browser: Return search form
    
    User->>Browser: Enter last name & submit
    Browser->>Controller: GET /owners?lastName=Smith
    Controller->>Service: findOwnersByLastName("Smith", pageable)
    Service->>Repo: findByLastNameContainingIgnoreCase("Smith", pageable)
    Repo->>DB: SELECT * FROM owners WHERE LOWER(last_name) LIKE '%smith%' LIMIT 5
    DB->>Repo: Return matching owners
    Repo->>Service: Return paginated owners
    Service->>Controller: Return search results
    
    alt Multiple Results Found
        Controller->>Browser: Display paginated results list
        User->>Browser: Click on specific owner
        Browser->>Controller: GET /owners/{ownerId}
        Controller->>Service: findOwnerById(ownerId)
        Service->>Repo: findById(ownerId)
        Repo->>DB: SELECT owner with pets and visits
        DB->>Repo: Return complete owner data
        Repo->>Service: Return owner entity
        Service->>Controller: Return owner
        Controller->>Browser: Display owner details
    else Single Result Found
        Controller->>Browser: Redirect to /owners/{ownerId}
    else No Results Found
        Controller->>Browser: Display "no owners found" message
    end
```

**Communication Patterns**:
- Case-insensitive search with database-level filtering
- Pagination with fixed page size (5 results per page)
- Automatic redirect for single-result searches
- Eager loading of related data for detailed views

---

### Workflow 5: Error Handling and Recovery

**Purpose**: Demonstrate system error handling and user experience
**Triggers**: User accesses error simulation endpoint or validation fails

```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant Controller as CrashController/OwnerController
    participant Service as ClinicService
    participant DB as Database
    participant ErrorPage as Error Page

    User->>Browser: Navigate to /oups (error simulation)
    Browser->>Controller: GET /oups
    Controller->>Controller: Throw simulated exception
    
    Note over Controller,ErrorPage: Spring Boot Error Handling Intercepts
    
    Controller->>Browser: Return error page (HTTP 500)
    Browser->>User: Display friendly error message
    
    User->>Browser: Submit invalid owner form (missing required fields)
    Browser->>Controller: POST /owners/new (invalid data)
    Controller->>Controller: Validation fails (@NotBlank violations)
    Controller->>Browser: Return form with field errors
    Browser->>User: Display form with validation messages
    
    User->>Browser: Submit corrected form
    Browser->>Controller: POST /owners/new (valid data)
    Controller->>Service: saveOwner(owner)
    Service->>DB: INSERT INTO owners
    DB->>Service: Constraint violation (duplicate data)
    Service->>Service: Catch DataAccessException
    Service->>Controller: Throw custom business exception
    Controller->>Browser: Return error page with specific message
    Browser->>User: Display business rule violation message
```

**Communication Patterns**:
- Global exception handling with Spring Boot
- Bean validation for input sanitization
- Database constraint violation handling
- Graceful degradation with user-friendly error messages
- Form re-display with validation feedback
```